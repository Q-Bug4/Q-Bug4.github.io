## AI 生成内容规则(强制)
1. **凡由 AI(agent)生成的文档/文章,一律视为草稿**,不允许进入正式发布流程:
   - AI 写完的正文只保存为草稿,由用户本人审阅、修改后手动发布
2. **必须用 Hexo 命令创建草稿**,不得手写文件路径:

   ```bash
   npx hexo new draft "标题"      # 生成 source/_drafts/标题.md
   ```

   - 草稿写作/修改期间用 `npx hexo server --drafts` 预览(不加盐不会出现在正式站点)
   - 人工审核通过后,由用户本人执行 `npx hexo publish "标题"` 把草稿移入 `_posts/`,
     **agent 不得执行 publish**
3. **AI 生成的文档必须打上 AI 标签并在开头声明来源**:
   - Front-matter 增加 `tags: [AI生成]`(已有标签则追加,不覆盖)
   - 正文第一行(front-matter 之后)插入声明块:

     ```markdown
     > 📝 本文由 AI agent 起草,尚未经过人工审核。发布前请由 Kael 本人审阅修改。
     > 撰写 agent:<agent 名称,如 Hermes / Claude Code / Codex> · 起草日期:YYYY-MM-DD
     ```

4. **发布动作仅限用户本人**:审核通过 → 删除 AI 声明块与 `AI生成` 标签
   (是否保留由用户决定)→ `hexo publish` → 确认 front-matter 完整
   (`permalink`/日期,hexo publish 会带入草稿创建时的日期)→ 推送 `source` 分支。
   **AI 不得执行 publish 和最后的推送。**

## Git / 部署规则(强制)

- 本仓库已在 GitHub 配置 Actions:**推送到 `source` 分支即自动构建并发布**,
  无需也不得运行 `hexo deploy`(deploy 是旧流程,已弃用)
- **任何 AI agent、脚本一律不得执行 `git push`**——包括但不限于:
  `git push`、`git push --force`、包装脚本、CI 触发、API 调用
- 本地 `git add` / `git commit` 允许,但只到提交为止;推送必须由用户本人手动执行
- 部署结果由用户推送后观察 GitHub Actions 运行状态确认,agent 不得代替
