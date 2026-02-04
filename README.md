# Power BI TMDL Manager - Claude Skill

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

A powerful skill for **Claude Code** that enables advanced management of Power BI Semantic Models using the TMDL (Tabular Model Definition Language) format. This skill allows you to document, scan, modify, and optimize Power BI models directly from the CLI.

## 🚀 Features

- **Auto-detection**: Automatically finds `.pbip` projects in your workspace.
- **Full Model Scanning**: Generates comprehensive documentation (JSON & Markdown) of your semantic models.
- **Bulk Operations**:
  - Create massive amount of measures (MoM, YoY, etc.) with a single command.
  - Reorganize measures into display folders.
  - Optimize DAX patterns.
  - Create calculated columns across multiple tables.
  - Auto-create relationships based on naming conventions.
- **TMDL Parsing**: Safely reads and edits TMDL files with correct indentation preservation.
- **Calculated Columns**: Full support for creating and managing DAX calculated columns.
- **Relationship Management**: Create, modify, delete, and analyze relationships between tables.

## 📦 Installation

1. Ensure you have **Claude Code** installed.
2. Navigate to your configuration directory (usually `~/.claude/skills` or your project's `.claude/skills` folder).
3. Copy the `powerbi-tmdl.md` file into that directory.

```bash
mkdir -p .claude/skills
cp powerbi-tmdl.md .claude/skills/
```

## 💡 Usage Examples

Once installed, you can ask Claude to perform tasks like:

- **"Document the current Power BI model"** → Generates reference docs.
- **"Create YoY and MoM measures for all Sales metrics"** → Bulk creates time intelligence measures.
- **"Optimize the DAX in the Measures table"** → Finds and fixes anti-patterns.
- **"Move all date-related measures to a 'Time Intelligence' folder"** → Reorganizes your model.
- **"Create a calculated column 'Age Group' in DimCustomers"** → Adds DAX calculated column.
- **"Create relationship between FactSales[CustomerID] and DimCustomers[CustomerID]"** → Adds relationship.
- **"Analyze all relationships in the model"** → Generates relationship analysis report.

## 🆕 What's New in v2.1.0

### Calculated Columns
- Complete syntax and workflow for creating DAX calculated columns
- Mass creation of time attribute columns (Year, Quarter, Month, etc.)
- Examples for classifications, flags, concatenations, and RELATED() usage
- Best practices guide for when to use columns vs measures

### Relationship Management
- Full CRUD operations for relationships (Create, Read, Update, Delete)
- Automatic cardinality detection
- Bulk relationship creation based on naming conventions
- Support for bidirectional and inactive relationships
- Relationship analysis with recommendations

### Enhanced Documentation
- Extended DAX function reference for calculated columns
- New display folder conventions
- Comprehensive workflow guides
- More practical examples

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Contributing

Contributions are welcome! Feel free to open issues or submit pull requests.

## 📧 Contact

Created by [@lsaucen](https://github.com/lsaucen)
