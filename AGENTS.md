# Working in packetloss99/somora_agent

Read [FORK-MAINTENANCE.md](FORK-MAINTENANCE.md) before changing, updating, releasing or deploying this fork.

- `main` is the maintained version, `upstream-stable` is an unmodified official release. `fork-upstream.json` records its provenance.
- All maintained-source changes and upstream updates go through a PR to this fork, never to upstream without a separate explicit request.
- Keep the Somora/Herdr patch small and separate from unrelated dependency updates. Never impersonate another agent type.
- Only the fork CI builds/tests here. No automatic publishing, deployment, npm publishing, Discord notifications or automatic dependency merges.
- Do not use the official updater on a fork deployment. Publish/install an explicitly approved, versioned fork artifact with checksum and rollback plan.
- Build success is not proof of live integration. Record what was actually tested; preserve configuration and conversation data.
