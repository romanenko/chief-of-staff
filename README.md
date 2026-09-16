# Chief of Staff

A Codex skill that uses a capable model to plan and review work, and more cost-efficient agents to carry out its detailed instructions.

1. Install [Codex](https://developers.openai.com/codex/cli), [Herdr](https://herdr.dev), [Worktrunk](https://worktrunk.dev), and [Hunk](https://www.hunk.dev).
2. Open your project in Herdr and start Codex. Choose a capable model with high reasoning. For example, **Astra at max** coordinates **Sol workers at medium**.
3. Paste this into Codex:

   > Install the chief-of-staff skill from https://github.com/romanenko/chief-of-staff. Also install Herdr’s skill if needed.

4. Then give it a task:

   > $chief-of-staff Improve the signup page and fix the broken links. Coordinate the work and show me the results.

The chief checks the risk before assigning work. Small, low-risk tasks return a pull request or finished file. Higher-risk tasks stay open beside a Hunk review, with notes explaining key choices and what needs your attention. The agent gets those review instructions before starting.

Stay in the main conversation to follow progress, or leave feedback directly in Hunk.
