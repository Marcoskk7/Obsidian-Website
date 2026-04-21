# Agent Framework Tool & Skill Implementation Deep Dive

> 四大框架：**agno** (Python) | **claude-code** (TypeScript) | **OpenHands** (Python) | **SimpleClaw** (Python)
>
> 生成时间：2026-04-21

---

## 目录

- [总览对比](#总览对比)
- [Part 1: agno](#part-1-agno)
- [Part 2: claude-code](#part-2-claude-code)
- [Part 3: OpenHands](#part-3-openhands)
- [Part 4: SimpleClaw](#part-4-simpleclaw)
- [架构洞察与总结](#架构洞察与总结)

---

## 总览对比

| 维度 | **agno** | **claude-code** | **OpenHands** | **SimpleClaw** |
|------|----------|-----------------|---------------|----------------|
| Tool 本质 | Python 函数/方法 (Pydantic BaseModel) | TypeScript 对象 (`buildTool` 工厂) | Action 数据类 (继承 Event) | Python 抽象类 |
| Skill 本质 | 知识包 (MD + scripts/ + references/) | Prompt 工作流 (MD + YAML frontmatter) | 遗留插件 (Python 模块, 已废弃) | Markdown 文档 (上下文注入) |
| 注册方式 | `@tool` 装饰器 / `Toolkit` 子类 | 集中式注册表 + feature flag | 隐式 (枚举 + 元组 dict comprehension) | `ToolRegistry` 字典 |
| 调度机制 | LLM 选择 -> model provider 执行 | LLM 选择 -> 权限管线 -> `call()` | LLM 选择 -> `getattr` 动态分派 | LLM 选择 -> Guard 验证 -> `execute()` |
| 并发模型 | 无显式并发管理 | 读操作并发(上限10)/写操作串行 | EventStream pub-sub | Guard 去重 + 延迟执行 |
| 权限/安全 | 审批工作流 (confirmation) | 规则引擎 + hooks + 分类器 (allow/deny/ask) | SecurityAnalyzer + confirmation 状态机 | Guard 命令拒绝列表 |
| MCP 支持 | 无 | 运行时动态组装 | 原生 MCPAction + MCPClient | MCPToolWrapper 包装 |

---

## Part 1: agno

### 1.1 Tool 核心: `Function` 类

**文件**: `agno/libs/agno/agno/tools/function.py`

`Function` 是 Pydantic BaseModel, 所有可调用工具的统一抽象:

```python
class Function(BaseModel):
    name: str
    description: Optional[str] = None
    parameters: Dict[str, Any] = Field(
        default_factory=lambda: {"type": "object", "properties": {}, "required": []},
    )
    strict: Optional[bool] = None
    instructions: Optional[str] = None
    add_instructions: bool = True
    entrypoint: Optional[Callable] = None
    skip_entrypoint_processing: bool = False
    show_result: bool = False
    stop_after_tool_call: bool = False
    pre_hook: Optional[Callable] = None
    post_hook: Optional[Callable] = None
    tool_hooks: Optional[List[Callable]] = None
    requires_confirmation: Optional[bool] = None
    requires_user_input: Optional[bool] = None
    user_input_fields: Optional[List[str]] = None
    external_execution: Optional[bool] = None
    approval_type: Optional[str] = None
    cache_results: bool = False
    cache_ttl: int = 3600

    # 运行时注入槽
    _agent: Optional[Any] = None
    _team: Optional[Any] = None
    _run_context: Optional[RunContext] = None
    _images: Optional[Sequence[Image]] = None
    _videos: Optional[Sequence[Video]] = None
    _audios: Optional[Sequence[Audio]] = None
    _files: Optional[Sequence[File]] = None
```

### 1.2 JSON Schema 自动生成: `process_entrypoint()`

从 Python 类型提示自动生成 OpenAI 兼容的 JSON Schema:

```python
def process_entrypoint(self, strict: bool = False):
    if self.skip_entrypoint_processing:
        if strict:
            self.process_schema_for_strict()
        return
    if self.entrypoint is None:
        return

    # 1. 内省签名和类型提示
    sig = signature(self.entrypoint)
    type_hints = get_type_hints(self.entrypoint)

    # 2. 剥离框架注入参数 (agent, team, run_context, 媒体类型)
    for name in ["agent", "team", "run_context", "images", "videos", "audios", "files"]:
        if name in sig.parameters and name in type_hints:
            del type_hints[name]

    # 3. 按类型剥离 -- 处理 `my_agent: Agent` 这种情况
    framework_types = (Agent, Team, RunContext, Image, Video, Audio, File)
    for param_name, hint in list(type_hints.items()):
        if isinstance(hint, type) and issubclass(hint, framework_types):
            del type_hints[param_name]
            excluded_params.append(param_name)

    # 4. 从 docstring 提取参数描述
    param_descriptions = {}
    if docstring := getdoc(self.entrypoint):
        parsed_doc = parse(docstring)
        for param in parsed_doc.params:
            param_descriptions[param.arg_name] = ...

    # 5. 通过 get_json_schema() 生成 JSON Schema
    parameters = get_json_schema(
        type_hints=param_type_hints,
        param_descriptions=param_descriptions,
        strict=strict,
    )

    # 6. 确定 required 字段
    if strict:
        parameters["required"] = [name for name in parameters["properties"] ...]
    else:
        parameters["required"] = [
            name for name, param in sig.parameters.items()
            if param.default == param.empty and name not in excluded_params
        ]

    # 7. 用 Pydantic validate_call 包装
    self.entrypoint = self._wrap_callable(self.entrypoint)
```

关键设计: 框架参数 (`agent`, `team`, `run_context`, 媒体) 从 schema 中剥离, 但在调用时通过 `_build_entrypoint_args()` 注入.

### 1.3 Tool 执行: `execute()`

```python
def execute(self) -> FunctionExecutionResult:
    self._handle_pre_hook()
    entrypoint_args = self._build_entrypoint_args()

    # 检查缓存
    if self.function.cache_results and not isgeneratorfunction(...):
        cached_result = self.function._get_cached_result(cache_file)
        if cached_result is not None:
            return FunctionExecutionResult(status="success", result=cached_result)

    try:
        # 如果有 tool_hooks, 构建嵌套中间件链
        if self.function.tool_hooks is not None:
            execution_chain = self._build_nested_execution_chain(entrypoint_args)
            result = execution_chain(self.function.name, self.function.entrypoint, self.arguments or {})
        else:
            result = self.function.entrypoint(**entrypoint_args, **self.arguments)

        if isgenerator(result):
            self.result = result  # 直接存储生成器
        else:
            self.result = result
            if self.function.cache_results:
                self.function._save_to_cache(cache_file, self.result)

        execution_result = FunctionExecutionResult(status="success", result=self.result, ...)
    except AgentRunException as e:
        ...
    finally:
        self._handle_post_hook()
    return execution_result
```

`_build_nested_execution_chain` 使用 `functools.reduce` 将 entrypoint 包裹在中间件链中, 每个 hook 接收 `next_func` 继续链条.

### 1.4 `@tool` 装饰器

**文件**: `agno/libs/agno/agno/tools/decorator.py`

```python
def tool(*args, **kwargs) -> Union[Function, Callable[[F], Function]]:
    def decorator(func: F) -> Function:
        # 根据函数类型选择包装器
        if isasyncgenfunction(func):
            wrapper = async_gen_wrapper
        elif _is_async_function(func):
            wrapper = async_wrapper
        else:
            wrapper = sync_wrapper

        update_wrapper(wrapper, func)

        # 检测 @approval 装饰器哨兵
        _approval_type = getattr(func, "_agno_approval_type", None)
        if _approval_type == "required":
            kwargs["requires_confirmation"] = True

        tool_config = {
            "name": kwargs.get("name", func.__name__),
            "description": kwargs.get("description", get_entrypoint_docstring(wrapper)),
            "entrypoint": wrapper,
            ...
        }
        function = Function(**tool_config)
        function.process_entrypoint()  # <-- 立即生成 JSON schema
        return function

    # 支持 @tool, @tool(), @tool(name="x") 三种用法
    if len(args) == 1 and callable(args[0]) and not kwargs:
        return decorator(args[0])
    return decorator
```

### 1.5 Toolkit 基类

**文件**: `agno/libs/agno/agno/tools/toolkit.py`

```python
def register(self, function: Union[Callable[..., Any], Function], name: Optional[str] = None) -> None:
    # 处理 Function 对象 (来自 @tool 装饰器)
    if isinstance(function, Function):
        is_async = function.entrypoint is not None and iscoroutinefunction(function.entrypoint)
        return self._register_decorated_tool(function, name, is_async=is_async)

    # 处理普通 callable -- 自动检测 async
    is_async = iscoroutinefunction(function)
    tool_name = name or function.__name__

    # include/exclude 过滤
    if self.include_tools is not None and tool_name not in self.include_tools:
        return
    if self.exclude_tools is not None and tool_name in self.exclude_tools:
        return

    f = Function(name=tool_name, entrypoint=function, ...)
    if is_async:
        self.async_functions[f.name] = f
    else:
        self.functions[f.name] = f

def get_async_functions(self) -> Dict[str, Function]:
    """Async 优先, 回退到 sync 实现."""
    merged = OrderedDict(self.functions)
    merged.update(self.async_functions)  # async 覆盖 sync
    return merged
```

### 1.6 Agent Tool 调度

**文件**: `agno/libs/agno/agno/agent/_tools.py`

#### `parse_tools()` -- 标准化异构工具输入

```python
def parse_tools(agent, tools, model, run_context=None, async_mode=False):
    _functions = []
    for tool in tools:
        if isinstance(tool, Dict):
            _functions.append(tool)  # 内置 model-provider tool

        elif isinstance(tool, Toolkit):
            toolkit_functions = tool.get_async_functions() if async_mode else tool.get_functions()
            for name, _func in toolkit_functions.items():
                _func = _func.model_copy(deep=True)
                _func._agent = agent                    # 注入 agent 引用
                _func._team = agent._team               # 注入 team 引用
                _func.process_entrypoint(strict=strict)  # 生成 JSON schema
                if agent.tool_hooks is not None:
                    _func.tool_hooks = agent.tool_hooks  # 附加 agent 级 hooks
                _functions.append(_func)

        elif isinstance(tool, Function):
            tool = tool.model_copy(deep=True)
            tool.process_entrypoint(strict=strict)
            tool._agent = agent
            _functions.append(tool)

        elif callable(tool):
            _func = Function.from_callable(tool, strict=strict)
            _func._agent = agent
            _functions.append(_func)
    return _functions
```

#### `run_tool()` -- 实际执行

```python
def run_tool(agent, run_response, run_messages, tool, functions=None, stream_events=False):
    agent.model = cast(Model, agent.model)
    function_call = agent.model.get_function_call_to_run_from_tool_execution(tool, functions)
    function_call_results: List[Message] = []

    for call_result in agent.model.run_function_call(
        function_call=function_call,
        function_call_results=function_call_results,
    ):
        if isinstance(call_result, ModelResponse):
            if call_result.event == ModelResponseEvent.tool_call_started.value:
                if stream_events:
                    yield create_tool_call_started_event(...)
            if call_result.event == ModelResponseEvent.tool_call_completed.value:
                tool.result = call_result.tool_executions[0].result
                ...
    run_messages.messages.extend(function_call_results)
```

### 1.7 Skill 系统

**文件**: `agno/libs/agno/agno/skills/skill.py`

```python
@dataclass
class Skill:
    name: str
    description: str
    instructions: str          # SKILL.md 正文
    source_path: str           # 技能目录路径
    scripts: List[str] = field(default_factory=list)
    references: List[str] = field(default_factory=list)
    metadata: Optional[Dict[str, Any]] = None
    license: Optional[str] = None
    compatibility: Optional[str] = None
    allowed_tools: Optional[List[str]] = None
```

**文件**: `agno/libs/agno/agno/skills/agent_skills.py`

`Skills` 编排器生成三个访问工具:

```python
def get_tools(self) -> List[Function]:
    tools = []
    tools.append(Function(
        name="get_skill_instructions",
        description="Load the full instructions for a skill.",
        entrypoint=self._get_skill_instructions,
    ))
    tools.append(Function(
        name="get_skill_reference",
        description="Load a reference document from a skill's references.",
        entrypoint=self._get_skill_reference,
    ))
    tools.append(Function(
        name="get_skill_script",
        description="Read or execute a script from a skill.",
        entrypoint=self._get_skill_script,
    ))
    return tools
```

Agent 中注入:
```python
if agent.skills is not None:
    agent_tools.extend(agent.skills.get_tools())
```

### 1.8 Hook 系统

**层1: 函数级 hooks** (`pre_hook` / `post_hook` / `tool_hooks`)

```python
def _handle_pre_hook(self):
    if self.function.pre_hook is not None:
        pre_hook_args = {}
        if "agent" in signature(self.function.pre_hook).parameters:
            pre_hook_args["agent"] = self.function._agent
        if "fc" in signature(self.function.pre_hook).parameters:
            pre_hook_args["fc"] = self
        self._safe_hook_call(self.function.pre_hook, pre_hook_args)
```

**层2: Agent 级 pre/post hooks** (`agno/agent/_hooks.py`)

```python
def execute_pre_hooks(agent, hooks, run_response, run_input, session, run_context, ...):
    all_args = {
        "run_input": run_input, "agent": agent, "session": session,
        "run_context": run_context, "user_id": user_id, ...
    }
    for hook in hooks:
        filtered_args = filter_hook_args(hook, all_args)  # 按签名过滤参数
        hook(**filtered_args)
```

Guardrail hooks 同步执行 (以便 `InputCheckError` 能传播), 非 guardrail hooks 缓冲到 `background_tasks`.

`BaseGuardrail` 实例归一化为 `.check` / `.async_check` 方法; `BaseEval` 实例用 `functools.partial` 包装.

---

## Part 2: claude-code

### 2.1 Tool 接口与 `buildTool` 工厂

**文件**: `claude-code/src/Tool.ts`

`Tool` 是一个 ~70 字段的泛型接口:

```typescript
export type Tool<
  Input extends AnyObject = AnyObject,
  Output = unknown,
  P extends ToolProgressData = ToolProgressData,
> = {
  aliases?: string[]
  searchHint?: string
  call(
    args: z.infer<Input>,
    context: ToolUseContext,
    canUseTool: CanUseToolFn,
    parentMessage: AssistantMessage,
    onProgress?: ToolCallProgress<P>,
  ): Promise<ToolResult<Output>>
  readonly inputSchema: Input              // Zod schema
  readonly inputJSONSchema?: ToolInputJSONSchema
  isConcurrencySafe(input: z.infer<Input>): boolean
  isEnabled(): boolean
  isReadOnly(input: z.infer<Input>): boolean
  isDestructive?(input: z.infer<Input>): boolean
  interruptBehavior?(): 'cancel' | 'block'
  checkPermissions(
    input: z.infer<Input>,
    context: ToolUseContext,
  ): Promise<PermissionResult>
  validateInput?(
    input: z.infer<Input>,
    context: ToolUseContext,
  ): Promise<ValidationResult>
  readonly name: string
  maxResultSizeChars: number
  readonly shouldDefer?: boolean
  readonly alwaysLoad?: boolean
  // ... rendering, prompt, mapping methods
}
```

`buildTool` 工厂函数填充 **fail-closed** 默认值:

```typescript
const TOOL_DEFAULTS = {
  isEnabled: () => true,
  isConcurrencySafe: (_input?: unknown) => false,  // 默认不并发安全
  isReadOnly: (_input?: unknown) => false,          // 默认假设有写操作
  isDestructive: (_input?: unknown) => false,
  checkPermissions: (input, _ctx?) =>
    Promise.resolve({ behavior: 'allow', updatedInput: input }),
  toAutoClassifierInput: (_input?: unknown) => '',
  userFacingName: (_input?: unknown) => '',
}

export function buildTool<D extends AnyToolDef>(def: D): BuiltTool<D> {
  return {
    ...TOOL_DEFAULTS,
    userFacingName: () => def.name,
    ...def,
  } as BuiltTool<D>
}
```

### 2.2 Tool 注册表

**文件**: `claude-code/src/tools.ts`

Feature flags 通过 `bun:bundle` 实现构建时 dead-code elimination:

```typescript
import { feature } from 'bun:bundle'

const REPLTool = process.env.USER_TYPE === 'ant'
  ? require('./tools/REPLTool/REPLTool.js').REPLTool : null
const SleepTool = feature('PROACTIVE') || feature('KAIROS')
  ? require('./tools/SleepTool/SleepTool.js').SleepTool : null
const cronTools = feature('AGENT_TRIGGERS')
  ? [CronCreateTool, CronDeleteTool, CronListTool] : []
const WebBrowserTool = feature('WEB_BROWSER_TOOL')
  ? require('./tools/WebBrowserTool/WebBrowserTool.js').WebBrowserTool : null
```

`assembleToolPool()` 合并内置工具与 MCP 工具:

```typescript
export function assembleToolPool(
  permissionContext: ToolPermissionContext,
  mcpTools: Tools,
): Tools {
  const builtInTools = getTools(permissionContext)
  const allowedMcpTools = filterToolsByDenyRules(mcpTools, permissionContext)
  // 分别排序以保持 API prompt cache 稳定性
  const byName = (a: Tool, b: Tool) => a.name.localeCompare(b.name)
  return uniqBy(
    [...builtInTools].sort(byName).concat(allowedMcpTools.sort(byName)),
    'name',
  )
}
```

内置工具排序为连续前缀, MCP 工具追加, 不会使 prompt cache key 失效.

### 2.3 Tool 执行管线

**文件**: `claude-code/src/services/tools/toolExecution.ts`

`runToolUse()` 是 async generator, 将 `ToolUseBlock` 解析为消息更新:

```typescript
export async function* runToolUse(
  toolUse: ToolUseBlock,
  assistantMessage: AssistantMessage,
  canUseTool: CanUseToolFn,
  toolUseContext: ToolUseContext,
): AsyncGenerator<MessageUpdateLazy, void> {
  let tool = findToolByName(toolUseContext.options.tools, toolName)
  // Fallback: 检查别名
  if (!tool) {
    const fallbackTool = findToolByName(getAllBaseTools(), toolName)
    if (fallbackTool && fallbackTool.aliases?.includes(toolName)) {
      tool = fallbackTool
    }
  }
  if (!tool) { /* yield error message */ return }
  if (toolUseContext.abortController.signal.aborted) { /* yield cancel */ return }

  for await (const update of streamedCheckPermissionsAndCallTool(...)) {
    yield update
  }
}
```

`checkPermissionsAndCallTool` 管线 (8步):

1. **Zod 验证** -- `tool.inputSchema.safeParse(input)`
2. **Tool 自定义验证** -- `tool.validateInput?.()`
3. **推测分类器** -- Bash 工具并行启动分类器
4. **PreToolUse hooks** -- `runPreToolUseHooks()`
5. **权限决策** -- `resolveHookPermissionDecision()` 合并 hook/规则/交互结果
6. **Tool 执行** -- `tool.call(callInput, context, canUseTool, ...)`
7. **PostToolUse hooks** -- `runPostToolUseHooks()`
8. **结果映射** -- `mapToolResultToToolResultBlockParam()` + `processToolResultBlock()`

Backfill-clone 模式防止可观察突变泄漏:

```typescript
let callInput = processedInput
const backfilledClone = tool.backfillObservableInput && typeof processedInput === 'object'
  ? ({ ...processedInput } as typeof processedInput)
  : null
if (backfilledClone) {
  tool.backfillObservableInput!(backfilledClone as Record<string, unknown>)
  processedInput = backfilledClone  // hooks/canUseTool 看到 backfilled 字段
}
// 权限决策后:
if (processedInput !== backfilledClone) {
  callInput = processedInput  // hook 替换了 input -- 使用新值
}
```

### 2.4 并发编排

**文件**: `claude-code/src/services/tools/toolOrchestration.ts`

工具按读写分批:

```typescript
function partitionToolCalls(
  toolUseMessages: ToolUseBlock[],
  toolUseContext: ToolUseContext,
): Batch[] {
  return toolUseMessages.reduce((acc: Batch[], toolUse) => {
    const tool = findToolByName(toolUseContext.options.tools, toolUse.name)
    const parsedInput = tool?.inputSchema.safeParse(toolUse.input)
    const isConcurrencySafe = parsedInput?.success
      ? (() => { try { return Boolean(tool?.isConcurrencySafe(parsedInput.data)) }
                 catch { return false } })()
      : false
    // 合并连续的并发安全块
    if (isConcurrencySafe && acc[acc.length - 1]?.isConcurrencySafe) {
      acc[acc.length - 1]!.blocks.push(toolUse)
    } else {
      acc.push({ isConcurrencySafe, blocks: [toolUse] })
    }
    return acc
  }, [])
}
```

并发执行, 可配置上限 (默认 10):

```typescript
async function* runToolsConcurrently(...): AsyncGenerator<MessageUpdateLazy, void> {
  yield* all(
    toolUseMessages.map(async function* (toolUse) {
      yield* runToolUse(toolUse, ...)
      markToolUseAsComplete(toolUseContext, toolUse.id)
    }),
    getMaxToolUseConcurrency(),  // 默认 10, 可通过环境变量配置
  )
}
```

### 2.5 Skill 系统

**文件**: `claude-code/src/tools/SkillTool/SkillTool.ts`

Skills 是 prompt 工作流, 通过 `SkillTool` 元工具调度. 两种执行模式:

#### Inline 模式 (默认) -- 注入上下文, 可覆盖权限和模型:

```typescript
async call({ skill, args }, context, canUseTool, parentMessage, onProgress?) {
  if (command?.type === 'prompt' && command.context === 'fork') {
    return executeForkedSkill(command, commandName, args, context, ...)
  }
  const processedCommand = await processPromptSlashCommand(commandName, args, commands, context)
  return {
    data: { success: true, commandName, allowedTools, model },
    newMessages,
    contextModifier(ctx) {
      let modifiedContext = ctx
      if (allowedTools.length > 0) {
        modifiedContext = { ...modifiedContext, getAppState() {
          return { ...previousGetAppState(), toolPermissionContext: {
            ...appState.toolPermissionContext,
            alwaysAllowRules: { ...rules, command: [...existing, ...allowedTools] }
          }}
        }}
      }
      if (model) {
        modifiedContext = { ...modifiedContext, options: {
          ...options, mainLoopModel: resolveSkillModelOverride(model, ctx.options.mainLoopModel)
        }}
      }
      return modifiedContext
    },
  }
}
```

#### Forked 模式 -- 独立子 Agent:

```typescript
async function executeForkedSkill(command, commandName, args, context, canUseTool, parentMessage, onProgress) {
  const { modifiedGetAppState, baseAgent, promptMessages, skillContent } =
    await prepareForkedCommandContext(command, args || '', context)
  for await (const message of runAgent({
    agentDefinition, promptMessages,
    toolUseContext: { ...context, getAppState: modifiedGetAppState },
    canUseTool, isAsync: false, querySource: 'agent:custom',
  })) { agentMessages.push(message) }
  return { data: { success: true, commandName, status: 'forked', agentId, result: extractResultText(agentMessages) } }
}
```

安全属性白名单 -- 只含安全属性的 skill 自动批准:

```typescript
const SAFE_SKILL_PROPERTIES = new Set([
  'type', 'progressMessage', 'model', 'effort', 'source', 'name',
  'description', 'isEnabled', 'aliases', 'getPromptForCommand', ...
])
function skillHasOnlySafeProperties(command: Command): boolean {
  for (const key of Object.keys(command)) {
    if (SAFE_SKILL_PROPERTIES.has(key)) continue
    const value = (command as Record<string, unknown>)[key]
    if (value === undefined || value === null) continue
    if (Array.isArray(value) && value.length === 0) continue
    return false
  }
  return true
}
```

### 2.6 权限系统

**文件**: `claude-code/src/types/permissions.ts`

```typescript
export type PermissionBehavior = 'allow' | 'deny' | 'ask'

export type PermissionAllowDecision<Input> = {
  behavior: 'allow'
  updatedInput?: Input
  userModified?: boolean
  decisionReason?: PermissionDecisionReason
}

export type PermissionAskDecision<Input> = {
  behavior: 'ask'
  message: string
  updatedInput?: Input
  suggestions?: PermissionUpdate[]
  pendingClassifierCheck?: PendingClassifierCheck
}

export type PermissionDenyDecision = {
  behavior: 'deny'
  message: string
  decisionReason: PermissionDecisionReason
}
```

决策来源 (discriminated union):

```typescript
export type PermissionDecisionReason =
  | { type: 'rule'; rule: PermissionRule }
  | { type: 'mode'; mode: PermissionMode }
  | { type: 'hook'; hookName: string; hookSource?: string; reason?: string }
  | { type: 'classifier'; classifier: string; reason: string }
  | { type: 'asyncAgent'; reason: string }
  | { type: 'sandboxOverride'; reason: 'excludedCommand' | 'dangerouslyDisableSandbox' }
  | { type: 'safetyCheck'; reason: string; classifierApprovable: boolean }
  | { type: 'permissionPromptTool'; permissionPromptToolName: string; toolResult: unknown }
  | { type: 'subcommandResults'; reasons: Map<string, PermissionResult> }
  | { type: 'workingDir'; reason: string }
  | { type: 'other'; reason: string }
```

### 2.7 Hook 系统

**文件**: `claude-code/src/services/tools/toolHooks.ts`

**PreToolUse hooks** -- 可 allow/deny/ask/passthrough:

```typescript
export async function* runPreToolUseHooks(
  toolUseContext, tool, processedInput, toolUseID, ...
): AsyncGenerator<
  | { type: 'hookPermissionResult'; hookPermissionResult: PermissionResult }
  | { type: 'hookUpdatedInput'; updatedInput: Record<string, unknown> }
  | { type: 'preventContinuation'; shouldPreventContinuation: boolean }
  | { type: 'stop' }
> {
  for await (const result of executePreToolHooks(tool.name, toolUseID, processedInput, ...)) {
    if (result.permissionBehavior !== undefined) {
      yield { type: 'hookPermissionResult', hookPermissionResult: {
        behavior: result.permissionBehavior, ...
      }}
    }
    if (result.updatedInput && result.permissionBehavior === undefined) {
      yield { type: 'hookUpdatedInput', updatedInput: result.updatedInput }
    }
  }
}
```

关键不变量: **hook 的 `allow` 不会绕过 settings.json 的 deny/ask 规则**:

```typescript
export async function resolveHookPermissionDecision(
  hookPermissionResult, tool, input, toolUseContext, canUseTool, assistantMessage, toolUseID,
) {
  if (hookPermissionResult?.behavior === 'allow') {
    const hookInput = hookPermissionResult.updatedInput ?? input
    // hook allow 跳过交互提示, 但 deny/ask 规则仍然生效
    const ruleCheck = await checkRuleBasedPermissions(tool, hookInput, toolUseContext)
    if (ruleCheck === null) return { decision: hookPermissionResult, input: hookInput }
    if (ruleCheck.behavior === 'deny') return { decision: ruleCheck, input: hookInput }
    return { decision: await canUseTool(tool, hookInput, ...), input: hookInput }
  }
  if (hookPermissionResult?.behavior === 'deny') return { decision: hookPermissionResult, input }
  return { decision: await canUseTool(tool, input, ...), input }
}
```

---

## Part 3: OpenHands

### 3.1 Action 基类与层次结构

**Base Event** (`openhands/events/event.py`):

```python
@dataclass
class Event:
    INVALID_ID = -1

    @property
    def id(self) -> int:
        if hasattr(self, '_id'):
            id_val = getattr(self, '_id')
            return int(id_val) if id_val is not None else Event.INVALID_ID
        return Event.INVALID_ID

    @property
    def source(self) -> EventSource | None:
        if hasattr(self, '_source'):
            src = getattr(self, '_source')
            return EventSource(src) if src is not None else None
        return None
```

私有 `_` 前缀属性通过 property 暴露, 避免污染序列化字段.

**Base Action** (`openhands/events/action/action.py`):

```python
class ActionConfirmationStatus(str, Enum):
    CONFIRMED = 'confirmed'
    REJECTED = 'rejected'
    AWAITING_CONFIRMATION = 'awaiting_confirmation'

class ActionSecurityRisk(int, Enum):
    UNKNOWN = -1
    LOW = 0
    MEDIUM = 1
    HIGH = 2

@dataclass
class Action(Event):
    runnable: ClassVar[bool] = False
```

**具体 Action 示例**:

```python
# CmdRunAction
@dataclass
class CmdRunAction(Action):
    command: str
    is_input: bool = False
    thought: str = ''
    blocking: bool = False
    hidden: bool = False
    action: str = ActionType.RUN
    runnable: ClassVar[bool] = True
    confirmation_state: ActionConfirmationStatus = ActionConfirmationStatus.CONFIRMED
    security_risk: ActionSecurityRisk = ActionSecurityRisk.UNKNOWN

# FileEditAction -- 支持 LLM-based 和 OH_ACI 两种模式
@dataclass
class FileEditAction(Action):
    path: str
    command: str = ''           # OH_ACI
    old_str: str | None = None  # OH_ACI str_replace
    new_str: str | None = None
    content: str = ''           # LLM-based
    start: int = 1
    end: int = -1
    action: str = ActionType.EDIT
    runnable: ClassVar[bool] = True
    impl_source: FileEditSource = FileEditSource.OH_ACI

# MCPAction
@dataclass
class MCPAction(Action):
    name: str
    arguments: dict[str, Any] = field(default_factory=dict)
    action: str = ActionType.MCP
    runnable: ClassVar[bool] = True
```

### 3.2 Action 注册与序列化

**文件**: `openhands/events/serialization/action.py`

```python
actions = (
    NullAction, CmdRunAction, IPythonRunCellAction, BrowseURLAction,
    BrowseInteractiveAction, FileReadAction, FileWriteAction, FileEditAction,
    AgentThinkAction, AgentFinishAction, AgentRejectAction, AgentDelegateAction,
    RecallAction, ChangeAgentStateAction, MessageAction, SystemMessageAction,
    CondensationAction, CondensationRequestAction, MCPAction, TaskTrackingAction,
    LoopRecoveryAction,
)

ACTION_TYPE_TO_CLASS = {action_class.action: action_class for action_class in actions}
```

Dict comprehension, 按每个类的 `action` 字符串字段做 key, O(1) 查找.

```python
def action_from_dict(action: dict) -> Action:
    if 'action' not in action:
        raise LLMMalformedActionError(f"'action' key is not found in {action=}")
    action_class = ACTION_TYPE_TO_CLASS.get(action['action'])
    if action_class is None:
        raise LLMMalformedActionError(
            f"'{action['action']=}' is not defined. Available: {ACTION_TYPE_TO_CLASS.keys()}"
        )
    args = action.get('args', {})
    if 'security_risk' in args and args['security_risk'] is not None:
        try:
            args['security_risk'] = ActionSecurityRisk(args['security_risk'])
        except (ValueError, TypeError):
            args.pop('security_risk')
    args = handle_action_deprecated_args(args)  # 向后兼容
    decoded_action = action_class(**args)
    return decoded_action
```

### 3.3 Runtime 动态分派

**文件**: `openhands/runtime/base.py`

```python
def run_action(self, action: Action) -> Observation:
    if not action.runnable:
        if isinstance(action, AgentThinkAction):
            return AgentThinkObservation('Your thought has been logged.')
        return NullObservation('')

    if (hasattr(action, 'confirmation_state')
        and action.confirmation_state == ActionConfirmationStatus.AWAITING_CONFIRMATION):
        return NullObservation('')

    action_type = action.action
    if action_type not in ACTION_TYPE_TO_CLASS:
        return ErrorObservation(f'Action {action_type} does not exist.')
    if not hasattr(self, action_type):
        return ErrorObservation(f'Action {action_type} is not supported in the current runtime.')

    if getattr(action, 'confirmation_state', None) == ActionConfirmationStatus.REJECTED:
        return UserRejectObservation('Action has been rejected by the user!')

    observation = getattr(self, action_type)(action)  # <-- 动态分派核心
    return observation
```

关键: `getattr(self, action_type)(action)` 将 action 类型字符串 (如 `"run"`, `"read"`) 解析为 runtime 实例上的方法.

### 3.4 Observation 系统

```python
# 基类
@dataclass
class Observation(Event):
    content: str

# 命令输出
@dataclass
class CmdOutputObservation(Observation):
    command: str
    observation: str = ObservationType.RUN
    metadata: CmdOutputMetadata = field(default_factory=CmdOutputMetadata)

    @property
    def exit_code(self) -> int:
        return self.metadata.exit_code

    @property
    def error(self) -> bool:
        return self.exit_code != 0

# MCP
@dataclass
class MCPObservation(Observation):
    observation: str = ObservationType.MCP
    name: str = ''
    arguments: dict[str, Any] = field(default_factory=dict)
```

### 3.5 MCP 集成

**文件**: `openhands/mcp/client.py`

```python
class MCPClientTool(Tool):
    def to_param(self) -> dict:
        return {
            'type': 'function',
            'function': {
                'name': self.name,
                'description': self.description,
                'parameters': self.inputSchema,
            },
        }

class MCPClient(BaseModel):
    client: Optional[Client] = None
    tools: list[MCPClientTool] = Field(default_factory=list)
    tool_map: dict[str, MCPClientTool] = Field(default_factory=dict)
    server_timeout: Optional[float] = None

    async def _initialize_and_list_tools(self) -> None:
        async with self.client:
            tools = await self.client.list_tools()
        for tool in tools:
            server_tool = MCPClientTool(
                name=tool.name, description=tool.description, inputSchema=tool.inputSchema,
            )
            self.tool_map[tool.name] = server_tool
            self.tools.append(server_tool)

    async def call_tool(self, tool_name: str, args: dict) -> CallToolResult:
        if tool_name not in self.tool_map:
            raise ValueError(f'Tool {tool_name} not found.')
        async with self.client:
            if self.server_timeout is not None:
                return await asyncio.wait_for(
                    self.client.call_tool_mcp(name=tool_name, arguments=args),
                    timeout=self.server_timeout,
                )
            else:
                return await self.client.call_tool_mcp(name=tool_name, arguments=args)
```

### 3.6 Agent Controller 循环

**文件**: `openhands/controller/agent_controller.py`

```python
async def _step(self) -> None:
    if self.get_agent_state() != AgentState.RUNNING:
        return
    if self._pending_action:
        return

    # 卡死检测
    if self.agent.config.enable_stuck_detection and self._is_stuck():
        await self._react_to_exception(AgentStuckInLoopError('Agent got stuck in a loop'))
        return

    action: Action = NullAction()
    if self._replay_manager.should_replay():
        action = self._replay_manager.step()
    else:
        action = self.agent.step(self.state)  # <-- LLM 调用
        action._source = EventSource.AGENT

    # 安全门控
    if action.runnable:
        if self.state.confirmation_mode and (
            type(action) is CmdRunAction
            or type(action) is IPythonRunCellAction
            or type(action) is BrowseInteractiveAction
            or type(action) is MCPAction
            ...
        ):
            await self._handle_security_analyzer(action)
            security_risk = getattr(action, 'security_risk', ActionSecurityRisk.UNKNOWN)
            if security_risk == ActionSecurityRisk.HIGH or (
                security_risk == ActionSecurityRisk.UNKNOWN and not self.security_analyzer
            ):
                action.confirmation_state = ActionConfirmationStatus.AWAITING_CONFIRMATION
        self._pending_action = action
```

### 3.7 Function Call 转换

**文件**: `openhands/llm/fn_call_converter.py`

为不支持原生 function calling 的模型提供文本格式转换:

```python
def convert_tool_call_to_string(tool_call: dict) -> str:
    ret = f'<function={tool_call["function"]["name"]}>\n'
    args = json.loads(tool_call['function']['arguments'])
    for param_name, param_value in args.items():
        ret += f'<parameter={param_name}>'
        if isinstance(param_value, list) or isinstance(param_value, dict):
            ret += json.dumps(param_value)
        else:
            ret += f'{param_value}'
        ret += '</parameter>\n'
    ret += '</function>'
    return ret
```

---

## Part 4: SimpleClaw

### 4.1 Tool 基类

**文件**: `SimpleClaw/simpleclaw/agent/tools/base.py`

```python
class Tool(ABC):
    _TYPE_MAP = {
        "string": str, "integer": int, "number": (int, float),
        "boolean": bool, "array": list, "object": dict,
    }

    @property
    @abstractmethod
    def name(self) -> str: ...

    @property
    @abstractmethod
    def description(self) -> str: ...

    @property
    @abstractmethod
    def parameters(self) -> dict[str, Any]: ...

    @abstractmethod
    async def execute(self, **kwargs: Any) -> str: ...

    def cast_params(self, params: dict[str, Any]) -> dict[str, Any]:
        """在验证前进行安全的 schema 驱动类型转换."""
        schema = self.parameters or {}
        if schema.get("type", "object") != "object":
            return params
        return self._cast_object(params, schema)

    def validate_params(self, params: dict[str, Any]) -> list[str]:
        """对参数进行 JSON Schema 验证, 返回错误列表."""
        if not isinstance(params, dict):
            return [f"parameters must be an object, got {type(params).__name__}"]
        schema = self.parameters or {}
        return self._validate(params, {**schema, "type": "object"}, "")

    def to_schema(self) -> dict[str, Any]:
        """转换为 OpenAI function schema 格式."""
        return {
            "type": "function",
            "function": {
                "name": self.name,
                "description": self.description,
                "parameters": self.parameters,
            },
        }
```

关键设计: `cast_params` 在 `validate_params` 之前运行, 所以 LLM 发送 `"42"` 给 integer 字段会被自动转换.

### 4.2 Tool Registry

**文件**: `SimpleClaw/simpleclaw/agent/tools/registry.py`

```python
class ToolRegistry:
    def __init__(self):
        self._tools: dict[str, Tool] = {}
        self._llm_visible: dict[str, bool] = {}

    def register(self, tool: Tool, *, expose_to_llm: bool = True) -> None:
        self._tools[tool.name] = tool
        self._llm_visible[tool.name] = expose_to_llm

    def get(self, name: str) -> Tool | None:
        return self._tools.get(name)

    def get_definitions(self) -> list[dict[str, Any]]:
        """获取所有 LLM 可见的 tool schema."""
        return [
            tool.to_schema()
            for name, tool in self._tools.items()
            if self._llm_visible.get(name, True)
        ]

    async def execute(self, name: str, params: dict[str, Any]) -> str:
        _HINT = "\n\n[Analyze the error above and try a different approach.]"
        tool = self._tools.get(name)
        if not tool:
            return f"Error: Tool '{name}' not found. Available: {', '.join(self.tool_names)}"
        try:
            params = tool.cast_params(params)
            errors = tool.validate_params(params)
            if errors:
                return f"Error: Invalid parameters for tool '{name}': " + "; ".join(errors) + _HINT
            result = await tool.execute(**params)
            if isinstance(result, str) and result.startswith("Error"):
                return result + _HINT
            return result
        except Exception as e:
            return f"Error executing {name}: {str(e)}" + _HINT
```

`_HINT` 后缀引导 LLM 自我纠正. `expose_to_llm=False` 用于隐藏底层 CronTool.

### 4.3 内置工具实现

#### EditFileTool -- 模糊匹配回退

**文件**: `SimpleClaw/simpleclaw/agent/tools/filesystem.py`

```python
def _find_match(content: str, old_text: str) -> tuple[str | None, int]:
    """精确匹配优先, 回退到行级空白无关滑动窗口."""
    if old_text in content:
        return old_text, content.count(old_text)
    # 滑动窗口: 去除每行空白后比较
    stripped_old = [l.strip() for l in old_text.splitlines()]
    for i in range(len(content_lines) - len(stripped_old) + 1):
        window = content_lines[i : i + len(stripped_old)]
        if [l.strip() for l in window] == stripped_old:
            candidates.append("\n".join(window))
    ...
```

匹配失败时, `_not_found_msg` 使用 `difflib.SequenceMatcher` 找到最佳部分匹配并显示 unified diff.

#### ExecTool -- 命令守卫

**文件**: `SimpleClaw/simpleclaw/agent/tools/shell.py`

```python
class ExecTool(Tool):
    def __init__(self, timeout=60, working_dir=None, deny_patterns=None, ...):
        self.deny_patterns = deny_patterns or [
            r"\brm\s+-[rf]{1,2}\b",       # rm -rf
            r"\bdel\s+/[fq]\b",           # Windows del
            r":\(\)\s*\{.*\};\s*:",        # fork bomb
        ]

    async def execute(self, command: str, working_dir=None, timeout=None, **kwargs) -> str:
        cwd = working_dir or self._working_dir_ctx.get() or os.getcwd()
        guard_error = self._guard_command(command, cwd)
        if guard_error:
            return guard_error
        process = await asyncio.create_subprocess_shell(command, ...)
        stdout, stderr = await asyncio.wait_for(process.communicate(), timeout=effective_timeout)
        # 头尾截断, 保留两端信息
        if len(result) > max_len:
            half = max_len // 2
            result = result[:half] + f"\n\n... ({len(result) - max_len:,} chars truncated) ...\n\n" + result[-half:]
```

#### SpawnTool -- 子 Agent 生成

```python
class SpawnTool(Tool):
    async def execute(self, task: str, label: str | None = None, **kwargs) -> str:
        return await self._manager_ctx.get().spawn(
            task=task, label=label,
            origin_channel=self._origin_channel_ctx.get(),
            origin_chat_id=self._origin_chat_id_ctx.get(),
            session_key=self._session_key_ctx.get(),
            tenant_key=self._tenant_key_ctx.get(),
        )
```

使用 `ContextVar` 实现多租户隔离.

### 4.4 Tool Execution Guard

**文件**: `SimpleClaw/simpleclaw/agent/tool_execution_guard.py`

```python
class ToolExecutionGuard:
    _WEB_SEARCH_TTL_SECONDS = 15.0
    _WEB_FETCH_TTL_SECONDS = 45.0
    _WEB_RETRY_DELAYS_SECONDS = (0.2, 0.5)  # 最多 3 次尝试
    _TRANSIENT_ERROR_MARKERS = ("timeout", "timed out", "429", "502", "503", "504", ...)

    def __init__(self, *, tools: ToolRegistry, prepare_deferred_action=None):
        self._handlers: dict[str, GuardHandler] = {}
        self._inflight: dict[str, asyncio.Task] = {}       # 在途去重
        self._recent_results: dict[str, tuple[float, ToolExecutionOutcome]] = {}  # TTL 缓存
        self.register_handler("web_search", self._execute_web_search)
        self.register_handler("web_fetch", self._execute_web_fetch)
```

**主入口** (`execute`):

```python
    async def execute(self, context: ToolExecutionContext) -> ToolExecutionOutcome:
        # 1. 检查延迟动作 (cron writes, HEARTBEAT.md, MEMORY.md)
        if self._prepare_deferred_action is not None:
            deferred = self._prepare_deferred_action(context.tool_name, dict(context.params))
            if deferred is not None:
                if context.deferred_actions is not None:
                    context.deferred_actions.append(deferred)
                return self._build_outcome(
                    tool_name=context.tool_name, ok=True, action="deferred",
                    content=deferred.synthetic_result, status="deferred",
                )
        # 2. 分派到注册处理器 (web 工具) 或默认管线
        handler = self._handlers.get(context.tool_name)
        if handler is not None:
            return await handler(context)
        return await self._execute_default_pipeline(context)
```

**去重** -- 防止同 turn 内重复 web 请求:

```python
    async def _execute_with_dedupe(self, context, *, ttl_seconds, runner):
        key = self._build_execution_key(context)  # tenant|session|lane|tool|params
        async with self._cache_lock:
            cached = self._recent_results.get(key)
            if cached is not None:
                return self._copy_outcome(cached_outcome, action="noop", status="recent_result_reused")
            task = self._inflight.get(key)
            owner = task is None
            if owner:
                task = asyncio.create_task(runner())
                self._inflight[key] = task
        outcome = await task
        # 成功结果缓存 TTL; 合并并发相同调用
```

**重试逻辑** -- 自动重试瞬态 web 错误:

```python
    async def _execute_with_retry(self, context, runner):
        attempts = len(self._WEB_RETRY_DELAYS_SECONDS) + 1  # 总共 3 次
        for attempt in range(1, attempts + 1):
            outcome = await runner()
            if outcome.ok or not self._is_retryable_web_outcome(outcome):
                return outcome
            delay = self._WEB_RETRY_DELAYS_SECONDS[attempt - 1]
            await asyncio.sleep(delay)
```

### 4.5 Agent Loop 与 Turn Processor

#### 工具注册

**文件**: `SimpleClaw/simpleclaw/agent/loop.py`

```python
def _register_default_tools(self) -> None:
    allowed_dir = self.workspace if self.restrict_to_workspace else None
    for cls in (ReadFileTool, WriteFileTool, EditFileTool, ListDirTool):
        self.tools.register(cls(
            workspace=self.workspace, allowed_dir=allowed_dir,
            document_store=self._document_store,
            memory_store=self._memory_store,
        ))
    self.tools.register(ExecTool(working_dir=str(self.workspace), ...))
    self.tools.register(WebSearchTool(config=self.web_search_config, proxy=self.web_proxy))
    self.tools.register(WebFetchTool(proxy=self.web_proxy))
    self.tools.register(MessageTool(send_callback=self.bus.publish_outbound))
    self.tools.register(SpawnTool(manager=self._create_subagent_manager(self.workspace)))
    if self.cron_service:
        cron_tool = CronTool(self.cron_service, ...)
        self.tools.register(cron_tool, expose_to_llm=False)  # 对 LLM 隐藏
        self.tools.register(CronAddOnceTool(cron_tool))       # 暴露子工具
        self.tools.register(CronAddIntervalTool(cron_tool))
```

#### 主循环

**文件**: `SimpleClaw/simpleclaw/agent/turn_processor.py`

```python
async def run_agent_loop(self, initial_messages, ...) -> tuple[str | None, list[str], list[dict]]:
    messages = initial_messages
    iteration = 0
    while iteration < self._max_iterations:
        iteration += 1
        tool_defs = self._filter_tool_definitions(self._tools.get_definitions(), excluded_tool_names)
        response = await self._get_llm_response(messages=messages, tool_defs=tool_defs, ...)

        if response.has_tool_calls:
            messages = builder.add_assistant_message(messages, response.content, tool_call_dicts, ...)
            for tool_call in response.tool_calls:
                # 通过 Guard 分派 (不是直接调 registry)
                outcome = await self._tool_execution_guard.execute(
                    ToolExecutionContext(
                        tool_name=tool_call.name,
                        params=tool_call.arguments,
                        deferred_actions=deferred_actions,
                        iteration=iteration,
                    )
                )
                messages = builder.add_tool_result(messages, tool_call.id, tool_call.name, outcome.content)
        else:
            final_content = self.strip_think(response.content)
            break
```

### 4.6 Skill 系统

**文件**: `SimpleClaw/simpleclaw/agent/skills.py`

```python
class SkillsLoader:
    def __init__(self, workspace, builtin_skills_dir=None, shared_workspace=None):
        self.workspace_skills = workspace / "skills"
        self.shared_skills = shared_workspace / "skills" if shared_workspace else None
        self.builtin_skills = builtin_skills_dir or BUILTIN_SKILLS_DIR

    def list_skills(self, filter_unavailable=True, *, include_builtin=False, source_filter=None):
        seen_names: set[str] = set()
        for source, root in locations:
            for skill_dir in sorted(root.iterdir()):
                skill_file = skill_dir / "SKILL.md"
                if skill_file.exists() and skill_dir.name not in seen_names:
                    seen_names.add(skill_dir.name)  # 先到先得优先级
                    skills.append({"name": skill_dir.name, "path": str(skill_file), "source": source})
        if filter_unavailable:
            return [s for s in skills if self._check_requirements(self._get_skill_meta(s["name"]))]

    def get_always_skills(self, source_filter=None) -> list[str]:
        """frontmatter 中 always=true 的技能 -- 自动注入每个 prompt."""
        result = []
        for s in self.list_skills(filter_unavailable=True, source_filter=source_filter):
            meta = self.get_skill_metadata(s["name"]) or {}
            skill_meta = self._parse_skill_metadata(meta.get("metadata", ""))
            if skill_meta.get("always") or meta.get("always"):
                result.append(s["name"])
        return result

    def load_skills_for_context(self, skill_names: list[str]) -> str:
        """加载并格式化技能用于 prompt 注入."""
        parts = []
        for name in skill_names:
            content = self.load_skill(name)
            if content:
                content = self._strip_frontmatter(content)
                content = self._strip_leading_heading(content)
                parts.append(f"### Skill: {name}\n\n{content}")
        return "\n\n---\n\n".join(parts)

    def _check_requirements(self, skill_meta: dict) -> bool:
        """检查 bins (shutil.which) 和环境变量 (os.environ.get)."""
        for b in requires.get("bins", []):
            if not shutil.which(b): return False
        for env in requires.get("env", []):
            if not os.environ.get(env): return False
        return True
```

### 4.7 MCP 集成

**文件**: `SimpleClaw/simpleclaw/agent/tools/mcp.py`

```python
class MCPToolWrapper(Tool):
    def __init__(self, session, server_name, tool_def, tool_timeout=30):
        self._session = session
        self._name = f"mcp_{server_name}_{tool_def.name}"  # 命名空间化避免冲突
        self._parameters = tool_def.inputSchema or {"type": "object", "properties": {}}

    async def execute(self, **kwargs) -> str:
        from mcp import types
        result = await asyncio.wait_for(
            self._session.call_tool(self._original_name, arguments=kwargs),
            timeout=self._tool_timeout,
        )
        parts = []
        for block in result.content:
            if isinstance(block, types.TextContent):
                parts.append(block.text)
            else:
                parts.append(str(block))
        return "\n".join(parts) or "(no output)"
```

三种传输自动检测:

```python
async def connect_mcp_servers(mcp_servers, registry, stack):
    for name, cfg in mcp_servers.items():
        if transport_type == "stdio":
            read, write = await stack.enter_async_context(stdio_client(params))
        elif transport_type == "sse":
            read, write = await stack.enter_async_context(sse_client(cfg.url, ...))
        elif transport_type == "streamableHttp":
            read, write, _ = await stack.enter_async_context(streamable_http_client(cfg.url, ...))

        session = await stack.enter_async_context(ClientSession(read, write))
        await session.initialize()
        for tool_def in (await session.list_tools()).tools:
            registry.register(MCPToolWrapper(session, name, tool_def, tool_timeout=cfg.tool_timeout))
```

---

## 架构洞察与总结

### Tool 定义模式对比

| 模式 | 框架 | 优点 | 缺点 |
|------|------|------|------|
| **Pydantic BaseModel + 装饰器** | agno | 类型提示自动生成 schema; 支持缓存/hooks/审批 | 重量级, Function 类 ~500 行 |
| **泛型接口 + 工厂函数** | claude-code | fail-closed 默认值; Zod schema 校验; 构建时 DCE | 接口 ~70 个字段, 复杂度高 |
| **数据类 + 动态分派** | OpenHands | 极简; 序列化友好; 事件驱动解耦 | 运行时方法名约定, 无编译期保证 |
| **抽象类 + Registry** | SimpleClaw | 简单直观; cast/validate 管线清晰 | 手写 JSON Schema, 无自动生成 |

### Skill 系统成熟度排序

1. **claude-code**: 最完整 -- inline/fork/remote 三种模式, 可覆盖模型和权限, `contextModifier` 机制
2. **agno**: 次之 -- 三个专用访问工具 (instructions/reference/script), 脚本可执行
3. **SimpleClaw**: 纯上下文注入, `always` 标记自动加载, 需求检查 (bins/env)
4. **OpenHands**: 遗留 v0, 已废弃 (2026-04-01 后计划移除)

### 安全模型对比

| 框架 | 权限决策 | 安全分析 | 命令守卫 |
|------|----------|----------|----------|
| **claude-code** | 多层 (rules + hooks + classifier + interactive ask) | 内置分类器 | Bash tool 级别 |
| **OpenHands** | SecurityAnalyzer + confirmation 状态机 (HIGH/UNKNOWN → 等待确认) | 可插拔 SecurityAnalyzer | 无显式命令拒绝 |
| **SimpleClaw** | Guard 层 + expose_to_llm 可见性控制 | 无 | 正则拒绝列表 (rm -rf, fork bomb) |
| **agno** | requires_confirmation + approval_type | 无 | 无 |

### 关键设计差异

1. **Schema 生成**: agno 从 Python 类型提示自动生成; claude-code 用 Zod; OpenHands 和 SimpleClaw 手写 JSON Schema
2. **执行隔离**: claude-code 的 Skill fork 模式最强 (独立子 Agent + token 预算); SimpleClaw 用 ContextVar 做多租户隔离
3. **错误引导**: SimpleClaw 的 `_HINT` 后缀和 EditFileTool 的 diff 展示最具特色, 主动引导 LLM 自我纠正
4. **并发控制**: 只有 claude-code 有显式的读写批次分离; SimpleClaw 的 Guard 去重解决了不同维度的问题
5. **向后兼容**: OpenHands 的 `handle_action_deprecated_args` 最完善; claude-code 用 tool aliases
