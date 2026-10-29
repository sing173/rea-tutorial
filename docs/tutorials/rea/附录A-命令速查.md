## 附录 A 命令速查

```bash
# 版本/就绪
rea --version
rea doctor --json
rea doctor --provider ghidra --json
rea capabilities --json

# 产物/静态
rea inspect-artifact PATH --json
rea inspect-go-binary PATH --detail summary --evidence_output FILE --json
rea analyze PATH --provider ghidra --json
rea function PATH --procedure NAME --provider ghidra --json
rea decompile PATH --procedure NAME --provider ghidra --json
rea xrefs PATH --provider ghidra --json
rea trace PATH --provider ghidra --json

# 运行时
rea capture-process SCENARIO.json --json
rea compare-process-captures A.json B.json --json

# 环境
rea environment-build SESSION.har OUT --name NAME --json
rea environment-serve MANIFEST.json --json
rea environment-export ...
rea environment-compare ...

# MCP
rea mcp
rea mcp --http

# 维护
rea update
rea uninstall
rea uninstall --purge-data
```

