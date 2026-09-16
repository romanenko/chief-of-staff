# Chief of Staff

A simple Codex skill implementing the **coordinator pattern**: one agent plans and reviews, while other agents make the changes.

Ideally, the chief uses a smart model with high reasoning effort. It gives workers detailed plans and chooses capable, faster, more cost-efficient models to carry them out.

## Install

Use [Codex](https://developers.openai.com/codex/cli) inside [Herdr](https://herdr.dev), with [Worktrunk](https://worktrunk.dev) and [Hunk](https://www.hunk.dev) installed. Then ask Codex:

> Install the chief-of-staff skill from https://github.com/romanenko/chief-of-staff. Also install Herdr’s skill if needed.

To start:

> $chief-of-staff Help me with [your task].
