# Quick review of opencode

## Context

- Test usage with locally runnable LLM (consumer grade)
- Choose opencode because: 1) open source, 2) TUI (Text based UI) - able to run in terminal + client/server option = more flexibilities of deployment scenario (eg call in CI pipeline, integration to webapp etc)

## Test case

- Model used: GLM 4.7 flash, REAPed, GGUF at ~Q3 (using unsloth UD)
- Basic RESTful CRUD webapp, book review as the entity, in memory DB, tech stack: fastapi, uv (require venv setup and modern practise). Request end to end test.

## Quirks, limitations, and warning found out

### General

- **Important** - there is a recent RCE vulnerability on opencode. For safety and due to the sensitive security posture of agentic coding softwares in general, strongly recommend to **never** run it outside of container or VM that has strong isolation.
- If using installation instead of docker pull (say, because you're in a cloud or local VM), recommend to use `pnpm`. Reason: `npm` have recent high severity software supply chain attack that caused widespread damage; one main attack vector is that `npm` trust the packages and run hooked post-install script automatically - if the package is comprimosed, that's basically full RCE (perhaps with user level privlege only, but still). `pnpm` as far as I know have stricter policy on this point.

### UX

- Nice to be able to get started immediately (zen mode with stealth mode free model) - but maybe could have have more explanation and also warning (eg the stealth model train on your data - this is mentioned in doc but not during on-boarding)
- Confusing UX with switching (sub)-agent (switching agent is not automatic - it's also still not clear to me from a quick test drive, how exactly does hand off work) - perhaps the LLM itself got confused too and the output it wrote is somewhat misleading to end user
- Slightly confusing UX on provider/connection (the in-app option menu allow some degree of config/setups, but it doesn't have full coverage - for our use case, we still need to edit config file outside the app, and the global config file does not exists - user need to read manual carefully, then carefully setup that file manually)

### Quirks and limitations

- **Context length config and compaction need special care for local model**: The max output config have special significance - opencode implicitly infer the trigger for automatic compaction based on the idea that we should compact when the next round of LLM output have potential to exceed context limit. But for local model, it is plausible to have max output = max context, then their equation will infer trigger line = when total output >= 0 tokens, i.e. immediately and always, in every round. This results in some very confusing behavior due to repetitive compaction. Moreover, it seems it is hardcoded that compaction will still preserve the most recent 40k tokens, so local models that tries to save VRAM by having total context < 40k => unrecoverable dead zone behavior? In short, it seems this feature have not considered the use case of resource constrained local LLM scenario.
- LSP integration maturity: LSP is nice to have indeed. For some reason, with my setup, it failed to pickup the venv and show import errors due to not installed packages that are in fact, already installed.
- Multi-edit maturity: experienced some failure on this tool, I have to ask the LLM to explicitly prefer single edit tool over it to recover (even though this would be inefficient)
- Shell access constraint: (Similar capability level to other TUI I tried - (shameless self promotion - I have referenced other people's code and used tmux to provide in my opinion a better shell access in my own agentic scaffold tutorial)) ephemeral shell, not enough control on long-running process or timeout control, need to do directory pinning by explicitly `cd <path> && <main command>`. Qwen seems well trained for this but GLM need a hint. GLM also reported that opencode tooling did not provide the full path (not even a privacy respecting, limited info, relative to user scope root path info, if security is a concern), resulting in an issue that it almost got stuck in until I give it hint (file name clash when path is masked resulting in "identity confusion").


## Recommendations

- Set the context length config carefully - you do not have to be completely honest about the true limits on the LLM inference engine side.
- Consider disabling compaction
- Consider disabling LSP, or tries to add manual path config to see if opencode LSP can pick up the venv successfully
- Provide carefully curated guidelines, both informational and behavioral, probably in `AGENTS.md`, to help LLM navigate the env more successfully.

