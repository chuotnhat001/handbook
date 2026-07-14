# Handbook Project — Session Context

## Status: in-progress
## Date: 2026-06-02
## Duration: ~3 hours

---

## Working on: Personal Knowledge Base — AI Engineering content expansion

### Summary

Handbook là personal knowledge base dạng static HTML (file:// protocol, no build tools). Session này tập trung vào cập nhật và mở rộng nhóm AI Engineering: update nội dung từ nguồn mới, tạo thêm 5 trang mới, thêm source-box cho tất cả trang, và tạo 7 mockups cho redesign trang chủ.

### Decisions Made

- Source box pattern: gradient background, icon link, nguồn tham khảo — áp dụng cho tất cả trang AI Engineering
- GSD Framework viết lại hoàn toàn theo tài liệu gsd-core (6-command loop thay vì 7-step sprint)
- GStack Sprint Playbook source link sửa sang github.com/garrytan/gstack
- Thứ tự cards trên trang chủ: Claude Code Handbook → Superpowers → GStack Sprint Playbook → GSD Framework → Skills Stack → GStack Sprint Practice → Multica → ECC → The Agency → Skills for Real Engineers → Spec Kit → Claude Code Best Practice
- Backup tại D:\ClaudeCode\projects\handbook_backup_20260601
- Mockups tạo trong thư mục mockups/ (7 mẫu A-G), chưa chọn mẫu cuối cùng

### Files Modified

**Updated:**
- index.html (cards order, new entries, Quick Reference)
- english/index.html (roadmap status, sidebar exercises)
- ai-engineering/claude-code-handbook.html (major update from claude-howto)
- ai-engineering/gstack-sprint-playbook.html (update from garrytan/gstack)
- ai-engineering/gsd-framework.html (full rewrite from open-gsd/gsd-core)
- ai-engineering/gstack-sprint-practice.html (source-box added)
- ai-engineering/multica.html (source-box added)
- ai-engineering/skills-stack.html (source-box added)
- ai-engineering/superpowers.html (source-box added)

**Created:**
- ai-engineering/ecc.html (from affaan-m/ECC)
- ai-engineering/agency-agents.html (from msitarzewski/agency-agents)
- ai-engineering/skills-real-engineers.html (from mattpocock/skills)
- ai-engineering/spec-kit.html (from github/spec-kit)
- ai-engineering/claude-code-best-practice.html (from shanraisshan/claude-code-best-practice)
- mockups/mockup-a-sidebar.html
- mockups/mockup-b-magazine.html
- mockups/mockup-c-compact.html
- mockups/mockup-d-bento.html
- mockups/mockup-e-notion.html
- mockups/mockup-f-terminal.html
- mockups/mockup-g-compact-grid.html

### Remaining Work

1. User chọn mockup trang chủ (A-G) hoặc kết hợp elements → apply vào index.html
2. Xóa thư mục mockups/ sau khi chọn xong
3. Có thể thêm nhóm mới (Programming, DevOps, Finance...)
4. English Giai đoạn 3 nếu cần mở rộng
5. Xem xét thêm trang Productivity (thêm tools khác ngoài Obsidian)

### Notes

- Project không dùng git — backup bằng folder copy
- Tất cả trang self-contained (inline CSS/JS), file:// compatible
- Template pattern: sidebar 260px + main content 860px, scroll-spy, copy-btn, source-box, callouts
- User muốn response bằng tiếng Việt, code/technical terms giữ English
