# agent-skills

Personal skills for coding agents, based on my day-to-day engineering and
research work.

## Skills

| Skill | Use |
| --- | --- |
| first-principles | Reason from outcomes and evidence; simplify through ablation |
| cpp-project-engineering | Apply detailed C++ project conventions |
| modern-cpp | Choose and use modern C++ language and library facilities |
| modern-cmake | Build clear target-based CMake projects |
| planning-clarification | Develop an idea into an actionable plan through focused questions |
| task-handoff | Keep compact task state for agent handoff and resumption |
| spec-driven-dev | Design, write, review, and implement development specs |
| use-subagents | Delegate useful independent work and assess its results |
| personal-markdown-note-writer | Turn source material into lasting Chinese notes |
| ai4science | Assist user-directed scientific analysis, implementation, and evaluation |
| pytorch | Build readable single-machine PyTorch implementations |

## Usage

Copy or link the desired directories from `skills/` into your agent's supported
skill location. Invoke a skill by name, or use automatic selection when the
runtime supports it.

For a C++ project, copy the [project AGENTS.md](skills/cpp-project-engineering/assets/AGENTS.md)
into a new repository, or merge its entry points into an existing `AGENTS.md`.
Add local contracts and configuration, such as directory layout, build presets,
dependency integration, and domain rules. The template routes to the shared
skills and keeps short fallbacks; detailed conventions stay in the skills.
