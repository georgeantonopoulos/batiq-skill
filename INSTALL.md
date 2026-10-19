# Install the BATIQ skill (instructions for an AI agent)

You are installing a skill that teaches you to drive the BATIQ compositor
through its command line. It is one folder with a `SKILL.md`. Do this:

1. **Pick the skills folder for the agent you are.** Install for the user
   (global), not just the current project, unless the user says otherwise.

   | Agent | Folder |
   | --- | --- |
   | Claude Code | `~/.claude/skills/batiq` |
   | OpenAI Codex | `~/.agents/skills/batiq` (older Codex builds: `~/.codex/skills/batiq`) |
   | Google Antigravity | `~/.gemini/antigravity/skills/batiq` |
   | Gemini CLI, Cursor, others that read Agent Skills | that tool's skills folder, else `~/.agents/skills/batiq` |

   On Windows, `~` is `%USERPROFILE%`.

2. **Clone it there** (a git checkout so it can be updated later):

   ```sh
   git clone --depth 1 https://github.com/georgeantonopoulos/batiq-skill.git ~/.claude/skills/batiq
   ```

   If the folder already exists, update it instead:
   `git -C <folder> pull --ff-only`.
   No git? Download
   https://github.com/georgeantonopoulos/batiq-skill/archive/refs/heads/main.zip
   and unzip its contents into the folder.

3. **Check it:** `<folder>/SKILL.md` exists and its front matter has `name: batiq`.

4. **Check BATIQ is installed** by looking for `batiq-headless` (see
   `SKILL.md`, section 2). If it is missing, tell the user it can be
   downloaded free from https://batiq.app/#download. Do not install it
   without asking.

5. **Tell the user** where the skill was installed, that your agent may need a
   restart (or a new session) to load it, and that you will use it whenever
   they ask you to work in BATIQ.

If your agent has no skills folder, read `SKILL.md` now and follow it for
this session.
