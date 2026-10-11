# AGENTS.md — 给 AI 助手的仓库须知

## npm 发布令牌的位置

- npm 发布令牌存放在 **Windows 用户级环境变量 `NPM_TOKEN`** 中（经 `setx` 持久化于注册表 `HKCU\Environment\NPM_TOKEN`；新开终端 / 登录会话自动可用）。
- `%USERPROFILE%\.npmrc` 已配置为引用该变量：

  ```ini
  //registry.npmjs.org/:_authToken=${NPM_TOKEN}
  ```

- 因此发布时**无需再传任何凭据**；在已打开的旧终端里若读不到该变量，重启终端即可。
- npm 账号：`smallsun`（`npm whoami` 可自检）。

## npm 发布流程（dsh-cost-meter）

1. 更新 `package.json` 版本号 + CHANGELOG；推送 master 并打 tag（安装链 install.ps1 / 双语 README 引用的 tag 需同步，参考既有 release commit）；
2. `npm pack --dry-run` 核对包内容后直接 `npm publish --access public`；
3. 认证失败（E401/E403）时按序检查：
   - PowerShell 执行 `[Environment]::GetEnvironmentVariable('NPM_TOKEN','User')` 确认变量存在且有效；
   - 失效则用新令牌重跑 `setx NPM_TOKEN <令牌>` 后重启终端再试。

## 安全约定

- **绝对不要**把令牌明文写入仓库文件、issue、PR、评论或命令输出——本仓库是公开的；
- 只允许记录指针位置（即本文件），令牌值只存在于用户环境变量中。

## Cloud Agent 开发

- 安装与验证命令以 README「开发与验证」为准：`pnpm install --frozen-lockfile`（pnpm **11.21.0**，与 `install.ps1` 的 `$PnpmVersion` 一致），然后 `node test/verify.mjs`。产物门禁是 `node scripts/build.mjs` 之后 `git diff --exit-code lib/client.js`。
- `test/verify.mjs` 调用 `zlib.zstdCompressSync`，需要 **Node.js ≥ 22.15**。Cloud Agent 镜像里的 `/exec-daemon/node`（22.14.0）没有该 API，回归会在历史回填处失败。环境把 `~/.nvm/versions/node/v22.23.3/bin` 放在 PATH 前面；`node -v` 应显示 v22.23.3。
- 本仓库没有必须常驻的开发服务器。本地余额模拟是 `node test/mock-balance.mjs`，监听 `127.0.0.1:3101` 的 `GET /user/balance`。
