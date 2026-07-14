# Handbook

Personal knowledge base. Static HTML, chạy trực tiếp từ file:// protocol.

## Cấu trúc

```
handbook/
├── index.html                              Trang chủ (search + card grid theo group)
├── ai-engineering/                         Group: AI Engineering
│   ├── claude-code-handbook.html
│   ├── gstack-sprint-practice.html
│   └── gstack-sprint-playbook.html
└── data/                                   Source files gốc (reference)
    ├── claude-code-handbook.html
    ├── gstack-sprint-practice-guide.html
    └── gstack_sprint_playbook.html
```

## Sử dụng

Mở `index.html` trong browser. Không cần server.

## Thêm trang mới

1. Tạo file HTML trong folder group (copy từ trang có sẵn làm template)
2. Thêm 1 entry vào array `groups` trong `index.html`:

```javascript
{ title: "Tên topic", url: "group-folder/file.html", desc: "Mô tả ngắn", icon: '<svg>...</svg>', color: "#hex" }
```

## Thêm group mới

1. Tạo folder mới (ví dụ: `programming/`)
2. Thêm object mới vào array `groups` trong `index.html`:

```javascript
{
  name: "Tên Group",
  topics: [
    { title: "...", url: "folder/file.html", desc: "...", icon: '<svg>...</svg>', color: "#hex" }
  ]
}
```

## Thiết kế

- Trang chủ: hero gradient, search realtime (case-insensitive, lọc theo title + desc), cards với accent color
- Trang con: self-contained (CSS inline), left menu theo nhóm nội dung, sticky Home button, copy button trên code blocks, scroll spy highlight active section
