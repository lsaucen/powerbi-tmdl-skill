# Power BI TMDL Manager - Claude Skill

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

A powerful skill for **Claude Code** that enables advanced management of Power BI Semantic Models using the TMDL (Tabular Model Definition Language) format. This skill allows you to document, scan, modify, and optimize Power BI models directly from the CLI.

## 🚀 Features

- **Auto-detection**: Automatically finds `.pbip` projects in your workspace.
- **Full Model Scanning**: Generates comprehensive documentation (JSON & Markdown) of your semantic models.
- **Bulk Operations**:
  - create massive amount of measures (MoM, YoY, etc.) with a single command.
  - Reorganize measures into display folders.
  - Optimize DAX patterns.
- **TMDL Parsing**: Safely reads and edits TMDL files with correct indentation preservation.

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

- **"Document the current Power BI model"** -> Generates reference docs.
- **"Create YoY and MoM measures for all Sales metrics"** -> Bulk creates time intelligence measures.
- **"Optimize the DAX in the Measures table"** -> Finds and fixes anti-patterns.
- **"Move all date-related measures to a 'Time Intelligence' folder"** -> Reorganizes your model.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
