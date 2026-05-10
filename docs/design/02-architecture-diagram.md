# 02-architecture-diagram.md

## OpenSpec CLI 架构图示意图

```mermaid
graph TD
    CLI --> CommandParser
    CLI --> CommandRegistry
    CLI --> UtilityServices
    CommandParser --> CommandHandler
    CommandRegistry --> CommandHandler
    UtilityServices --> FileOperations
    UtilityServices --> ConfigManagement
    UtilityServices --> OutputFormatting
    CommandParser --> Telemetry
    CommandRegistry --> Telemetry
    UtilityServices --> Telemetry

    subgraph CLI
        CommandParser
        CommandRegistry
        UtilityServices
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