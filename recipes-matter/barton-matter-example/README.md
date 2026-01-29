When updating Matter versions, you may need to regnerate
zap and matter files. To do so:

1. Clone matter and checkout the version you are upgrading to.
2. Source scripts/bootstrap.sh.
3. Copy your zap file into the root of the repo.
4. Run ./scripts/tools/zap/run_zaptool.sh and open your zap file.
5. (Optional) Make edits to your zap file.
6. Save your zap file. This should automatically include any version upgrade changes needed by the SDK itself (see: "upgradeRules" of the zcl.json file).
7. Run ./scripts/tools/zap/generate.py <zapfile> to generate your matter file.