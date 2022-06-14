- 👋 Hi, I’m @jack-hanblue
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
jack-hanblue/jack-hanblue is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
name: Autoclose
on:
  pull_request_target:
    branches:
      - 'main'
    paths:
      - '.gitignore'

jobs:
  autoclose:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/github-script@7a5c598405937d486b0331594b5da2b14db670da # tag=v6
        with:
          github-token: ${{secrets.GITHUB_TOKEN}}
          script: |
            const patches = [
              "@@ -63,7 +63,6 @@ $RECYCLE.BIN/\n # Icon must end with two \\r\n Icon\n \n-\n # Thumbnails\n ._*\n ",
              "@@ -63,7 +63,6 @@ $RECYCLE.BIN/\n # Icon must end with two \\r\n Icon\n \n-\n # Thumbnails\n ._*\n \n@@ -165,7 +164,6 @@ config/curriculum.json\n config/i18n/all-langs.js\n config/certification-settings.js\n \n-\n ### vim ###\n # Swap\n [._]*.s[a-v][a-z]",
              "@@ -165,7 +165,6 @@ config/curriculum.json\n config/i18n/all-langs.js\n config/certification-settings.js\n \n-\n ### vim ###\n # Swap\n [._]*.s[a-v][a-z]"
            ];
            const files = await github.rest.pulls.listFiles({
              owner: context.payload.repository.owner.login,
              repo: context.payload.repository.name,
              pull_number: context.payload.pull_request.number,
            });
            if (
              files.data.length === 1 &&
              files.data[0].filename === ".gitignore" &&
              patches.includes(files.data[0].patch)
            ) {
              core.setFailed("Invalid PR detected.");
              github.rest.issues.createComment({
                issue_number: context.issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                body: "Thank you for opening this pull request.\n\nThis is a standard message notifying you that we've reviewed your pull request and have decided not to merge it. We would welcome future pull requests from you.\n\nThank you and happy coding.",
              });
              await github.rest.pulls.update({
                owner: context.payload.repository.owner.login,
                repo: context.payload.repository.name,
                pull_number: context.payload.pull_request.number,
                state: "closed"
              });
              await github.rest.issues.addLabels({
                owner: context.payload.repository.owner.login,
                repo: context.payload.repository.name,
                issue_number: context.issue.number,
                labels: ["invalid"]
              });
            }
