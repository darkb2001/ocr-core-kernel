# Tree Repo — ocr-core-kernel

Đọc file này trước. Tạo toàn bộ cấu trúc thư mục và file rỗng bên dưới, sau đó mới điền nội dung từng file.

## Hướng dẫn tạo cấu trúc

Chạy lệnh sau để tạo tất cả thư mục và file rỗng trong một lượt:

```bash
mkdir -p ocr-core-kernel/{algorithms/{agent,diff,llmloop,rules,template,tool},allowlist,design,prompts,rules/rule_docs,specs}

touch ocr-core-kernel/ARCHITECTURE.md
touch ocr-core-kernel/CURSOR-INTEGRATION.md
touch ocr-core-kernel/FILE_INDEX.txt
touch ocr-core-kernel/PIPELINE.md
touch ocr-core-kernel/README.md
touch ocr-core-kernel/REBUILD-CHECKLIST.md
touch ocr-core-kernel/tree-repo.md

touch ocr-core-kernel/algorithms/README.md
touch ocr-core-kernel/algorithms/agent/grouping.go
touch ocr-core-kernel/algorithms/agent/selection.go
touch ocr-core-kernel/algorithms/diff/hunk.go
touch ocr-core-kernel/algorithms/diff/parser.go
touch ocr-core-kernel/algorithms/diff/relocation.go
touch ocr-core-kernel/algorithms/diff/resolver.go
touch ocr-core-kernel/algorithms/llmloop/compression.go
touch ocr-core-kernel/algorithms/llmloop/loop.go
touch ocr-core-kernel/algorithms/llmloop/tool_failure_streak.go
touch ocr-core-kernel/algorithms/rules/system_rules.go
touch ocr-core-kernel/algorithms/template/template.go
touch ocr-core-kernel/algorithms/tool/code_comment.go
touch ocr-core-kernel/algorithms/tool/comment_args_repair.go
touch ocr-core-kernel/algorithms/tool/comment_collector.go
touch ocr-core-kernel/algorithms/tool/definitions.go

touch ocr-core-kernel/allowlist/allowed_ext.go
touch ocr-core-kernel/allowlist/default_exclude_patterns.json
touch ocr-core-kernel/allowlist/default_secret_patterns.json
touch ocr-core-kernel/allowlist/secret_path.go
touch ocr-core-kernel/allowlist/supported_file_types.json

touch ocr-core-kernel/design/comment-positioning.md
touch ocr-core-kernel/design/hard-constraints.md
touch ocr-core-kernel/design/prompt-system.md
touch ocr-core-kernel/design/what-was-excluded.md

touch ocr-core-kernel/prompts/grouping_task_system.md
touch ocr-core-kernel/prompts/grouping_task_user.md
touch ocr-core-kernel/prompts/main_task_system.md
touch ocr-core-kernel/prompts/main_task_user.md
touch ocr-core-kernel/prompts/memory_compression_task_system.md
touch ocr-core-kernel/prompts/memory_compression_task_user.md
touch ocr-core-kernel/prompts/plan_task_system.md
touch ocr-core-kernel/prompts/plan_task_user.md
touch ocr-core-kernel/prompts/re_location_task_system.md
touch ocr-core-kernel/prompts/re_location_task_user.md
touch ocr-core-kernel/prompts/review_filter_task_system.md
touch ocr-core-kernel/prompts/review_filter_task_user.md

touch ocr-core-kernel/rules/rule_docs/arkts.md
touch ocr-core-kernel/rules/rule_docs/astro.md
touch ocr-core-kernel/rules/rule_docs/bicep.md
touch ocr-core-kernel/rules/rule_docs/build_gradle.md
touch ocr-core-kernel/rules/rule_docs/c.md
touch ocr-core-kernel/rules/rule_docs/capnp.md
touch ocr-core-kernel/rules/rule_docs/cargo_toml.md
touch ocr-core-kernel/rules/rule_docs/composer_json.md
touch ocr-core-kernel/rules/rule_docs/cpp.md
touch ocr-core-kernel/rules/rule_docs/default.md
touch ocr-core-kernel/rules/rule_docs/elm.md
touch ocr-core-kernel/rules/rule_docs/freemarker.md
touch ocr-core-kernel/rules/rule_docs/github_config.md
touch ocr-core-kernel/rules/rule_docs/github_workflows.md
touch ocr-core-kernel/rules/rule_docs/go.md
touch ocr-core-kernel/rules/rule_docs/graphql.md
touch ocr-core-kernel/rules/rule_docs/handlebars_mustache.md
touch ocr-core-kernel/rules/rule_docs/haskell.md
touch ocr-core-kernel/rules/rule_docs/java.md
touch ocr-core-kernel/rules/rule_docs/json.md
touch ocr-core-kernel/rules/rule_docs/jsonnet.md
touch ocr-core-kernel/rules/rule_docs/julia.md
touch ocr-core-kernel/rules/rule_docs/kotlin.md
touch ocr-core-kernel/rules/rule_docs/mapper_dao_xml.md
touch ocr-core-kernel/rules/rule_docs/matlab.md
touch ocr-core-kernel/rules/rule_docs/nim.md
touch ocr-core-kernel/rules/rule_docs/nix.md
touch ocr-core-kernel/rules/rule_docs/objc.md
touch ocr-core-kernel/rules/rule_docs/ocaml.md
touch ocr-core-kernel/rules/rule_docs/package_json.md
touch ocr-core-kernel/rules/rule_docs/php.md
touch ocr-core-kernel/rules/rule_docs/po.md
touch ocr-core-kernel/rules/rule_docs/pom_xml.md
touch ocr-core-kernel/rules/rule_docs/pot.md
touch ocr-core-kernel/rules/rule_docs/prisma.md
touch ocr-core-kernel/rules/rule_docs/properties.md
touch ocr-core-kernel/rules/rule_docs/protobuf.md
touch ocr-core-kernel/rules/rule_docs/pug.md
touch ocr-core-kernel/rules/rule_docs/python.md
touch ocr-core-kernel/rules/rule_docs/r.md
touch ocr-core-kernel/rules/rule_docs/rego.md
touch ocr-core-kernel/rules/rule_docs/rust.md
touch ocr-core-kernel/rules/rule_docs/solidity.md
touch ocr-core-kernel/rules/rule_docs/swift.md
touch ocr-core-kernel/rules/rule_docs/terraform.md
touch ocr-core-kernel/rules/rule_docs/thrift.md
touch ocr-core-kernel/rules/rule_docs/ts_js_tsx_jsx.md
touch ocr-core-kernel/rules/rule_docs/verilog.md
touch ocr-core-kernel/rules/rule_docs/vhdl.md
touch ocr-core-kernel/rules/rule_docs/vyper.md
touch ocr-core-kernel/rules/rule_docs/yaml.md
touch ocr-core-kernel/rules/rule_docs/zig.md

touch ocr-core-kernel/specs/README.md
touch ocr-core-kernel/specs/filter_tools.json
touch ocr-core-kernel/specs/model_diff.go
touch ocr-core-kernel/specs/model_review.go
touch ocr-core-kernel/specs/prompt_serialization.md
touch ocr-core-kernel/specs/runtime_thresholds.md
touch ocr-core-kernel/specs/system_rules.json
touch ocr-core-kernel/specs/task_template.json
touch ocr-core-kernel/specs/tools.json
touch ocr-core-kernel/specs/user_rule_format.json
```

## Cây thư mục đầy đủ (106 files)

```
ocr-core-kernel/
├── ARCHITECTURE.md
├── CURSOR-INTEGRATION.md
├── FILE_INDEX.txt
├── PIPELINE.md
├── README.md
├── REBUILD-CHECKLIST.md
├── tree-repo.md
│
├── algorithms/
│   ├── README.md
│   ├── agent/
│   │   ├── grouping.go
│   │   └── selection.go
│   ├── diff/
│   │   ├── hunk.go
│   │   ├── parser.go
│   │   ├── relocation.go
│   │   └── resolver.go
│   ├── llmloop/
│   │   ├── compression.go
│   │   ├── loop.go
│   │   └── tool_failure_streak.go
│   ├── rules/
│   │   └── system_rules.go
│   ├── template/
│   │   └── template.go
│   └── tool/
│       ├── code_comment.go
│       ├── comment_args_repair.go
│       ├── comment_collector.go
│       └── definitions.go
│
├── allowlist/
│   ├── allowed_ext.go
│   ├── default_exclude_patterns.json
│   ├── default_secret_patterns.json
│   ├── secret_path.go
│   └── supported_file_types.json
│
├── design/
│   ├── comment-positioning.md
│   ├── hard-constraints.md
│   ├── prompt-system.md
│   └── what-was-excluded.md
│
├── prompts/
│   ├── grouping_task_system.md
│   ├── grouping_task_user.md
│   ├── main_task_system.md
│   ├── main_task_user.md
│   ├── memory_compression_task_system.md
│   ├── memory_compression_task_user.md
│   ├── plan_task_system.md
│   ├── plan_task_user.md
│   ├── re_location_task_system.md
│   ├── re_location_task_user.md
│   ├── review_filter_task_system.md
│   └── review_filter_task_user.md
│
├── rules/
│   └── rule_docs/
│       ├── arkts.md
│       ├── astro.md
│       ├── bicep.md
│       ├── build_gradle.md
│       ├── c.md
│       ├── capnp.md
│       ├── cargo_toml.md
│       ├── composer_json.md
│       ├── cpp.md
│       ├── default.md
│       ├── elm.md
│       ├── freemarker.md
│       ├── github_config.md
│       ├── github_workflows.md
│       ├── go.md
│       ├── graphql.md
│       ├── handlebars_mustache.md
│       ├── haskell.md
│       ├── java.md
│       ├── json.md
│       ├── jsonnet.md
│       ├── julia.md
│       ├── kotlin.md
│       ├── mapper_dao_xml.md
│       ├── matlab.md
│       ├── nim.md
│       ├── nix.md
│       ├── objc.md
│       ├── ocaml.md
│       ├── package_json.md
│       ├── php.md
│       ├── po.md
│       ├── pom_xml.md
│       ├── pot.md
│       ├── prisma.md
│       ├── properties.md
│       ├── protobuf.md
│       ├── pug.md
│       ├── python.md
│       ├── r.md
│       ├── rego.md
│       ├── rust.md
│       ├── solidity.md
│       ├── swift.md
│       ├── terraform.md
│       ├── thrift.md
│       ├── ts_js_tsx_jsx.md
│       ├── verilog.md
│       ├── vhdl.md
│       ├── vyper.md
│       ├── yaml.md
│       └── zig.md
│
└── specs/
    ├── README.md
    ├── filter_tools.json
    ├── model_diff.go
    ├── model_review.go
    ├── prompt_serialization.md
    ├── runtime_thresholds.md
    ├── system_rules.json
    ├── task_template.json
    ├── tools.json
    └── user_rule_format.json
```

## Mô tả ngắn từng nhóm

| Thư mục / File | Nội dung |
|---|---|
| `README.md` | Điểm vào, thứ tự đọc |
| `ARCHITECTURE.md` | Triết lý + sơ đồ component |
| `PIPELINE.md` | Pipeline runtime từng bước |
| `REBUILD-CHECKLIST.md` | Backlog implement |
| `CURSOR-INTEGRATION.md` | Cursor Skill + delegate mode |
| `prompts/` | 12 prompt templates (system + user cho 6 tasks) |
| `specs/tools.json` | 6 tool schemas (main phase) |
| `specs/filter_tools.json` | 2 tool schemas (filter phase) |
| `specs/task_template.json` | Wiring task + numeric thresholds |
| `specs/runtime_thresholds.md` | Tất cả hằng số runtime (60%/80%/10/100...) |
| `specs/prompt_serialization.md` | Format chính xác của mọi `{{placeholder}}` |
| `specs/system_rules.json` | Map glob → rule doc |
| `specs/user_rule_format.json` | Schema rule.json cho user |
| `specs/model_diff.go` | Struct `Diff` |
| `specs/model_review.go` | Struct `LlmComment` |
| `algorithms/` | Go reference implementations (không chạy được, chỉ để đọc) |
| `rules/rule_docs/` | 52 checklist theo ngôn ngữ |
| `allowlist/` | Ext được review, secret paths, default excludes |
| `design/` | Giải thích tại sao mỗi hard constraint tồn tại |
