# 04-execution-flow.md

## OpenSpec CLI 执行流程图

```mermaid
graph LR
    CLI -- Command Parsing --> CommandMatching
    CommandMatching -- Command Execution --> ExecutionSetup
    ExecutionSetup -- Command Execution --> CommandExecution
    CommandExecution -- Error Handling --> Telemetry
    Telemetry -- Command Completion --> CommandCompletion

    subgraph CLI
        CommandParsing
        CommandMatching
        ExecutionSetup
        CommandExecution
        ErrorHandling
        Telemetry
        CommandCompletion
    end

    subgraph CommandModules
        CommandHandler
    end

    subgraph UtilityModules
        FileOperations
        ConfigManagement
        OutputFormatting
    end

    subgraph Telemetry
        Telemetry
    end

    style CLI fillcolor=#FFD700,strokecolor=#FFA500
    style CommandModules fillcolor=#98FB98,strokecolor=#00FF00
    style UtilityModules fillcolor=#ADD8E6,strokecolor=#0000FF
    style Telemetry fillcolor=#FFB6C1,strokecolor=#FF69B4
```