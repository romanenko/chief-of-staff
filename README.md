# Chief of Staff

A simple Codex skill implementing the **coordinator pattern**: one agent plans and reviews, while other agents make the changes.

The chief gives workers detailed plans and reviews their results. Defaults are Astra with medium reasoning for the chief and faster, more cost-efficient Sol with medium reasoning for workers. They communicate directly through Codex; Herdr organizes their workspaces and reviews.

## Install

Use [Codex](https://developers.openai.com/codex/cli) inside [Herdr](https://herdr.dev), with [Worktrunk](https://worktrunk.dev) and [Hunk](https://www.hunk.dev) installed. Then ask Codex:

> Install the chief-of-staff skill from https://github.com/romanenko/chief-of-staff. Also install Herdr’s skill if needed.

To start:

> $chief-of-staff Help me with [your task].
