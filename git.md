# Git Command & Git Flow

## Git Flow

### Mô hình branch

- `main` — code production, luôn ổn định
- `develop` — branch tích hợp, nơi các feature merge vào
- `feature/ten-tinh-nang` — phát triển tính năng mới, tách từ `develop`
- `hotfix/ten-loi` — vá lỗi khẩn trên production, tách từ `main`

### Luồng làm việc hàng ngày

1. Tạo branch mới từ `develop`
2. Code, commit thường xuyên với message rõ ràng
3. Push lên remote, tạo Pull Request / Merge Request vào `develop`
4. Review & merge
5. Khi release: merge `develop` → `main`, tag version

### Luồng hotfix

1. Tách branch `hotfix/...` từ `main`
2. Fix xong → merge vào cả `main` và `develop`
3. Tag lại version trên `main`

---

## Các lệnh Git thường dùng

### Khởi tạo & cấu hình

Dùng khi bắt đầu một project mới hoặc thiết lập thông tin cá nhân cho Git trên máy.

```bash
git init                        # khởi tạo repo mới trong thư mục hiện tại
git clone <url>                 # clone repo từ remote về local
git config --global user.name "Name"        # đặt tên tác giả cho mọi commit
git config --global user.email "email@example.com"  # đặt email tác giả
```

### Branch

Quản lý branch — tạo, chuyển, xóa. Mỗi tính năng nên có branch riêng, tránh làm thẳng trên `main`/`develop`.

```bash
git branch                      # xem danh sách branch, branch hiện tại có dấu *
git branch <name>               # tạo branch mới nhưng chưa chuyển sang
git checkout <name>             # chuyển sang branch đã có
git checkout -b <name>          # tạo branch mới và chuyển sang luôn
git branch -d <name>            # xóa branch đã được merge
git branch -D <name>            # ép xóa branch dù chưa merge
```

### Làm việc với thay đổi

Chu trình cơ bản: chỉnh file → stage → commit. Nên commit nhỏ, thường xuyên, message rõ ràng.

```bash
git status                      # xem file nào đang thay đổi, đã stage hay chưa
git diff                        # xem chi tiết thay đổi chưa được stage
git diff --staged               # xem thay đổi đã stage, chuẩn bị commit
git add <file>                  # stage một file cụ thể
git add .                       # stage tất cả thay đổi trong thư mục hiện tại
git commit -m "message"         # tạo commit với message
git commit --amend              # sửa commit cuối: đổi message hoặc thêm file bị quên
```

### Remote

Đồng bộ code giữa local và remote (GitHub, GitLab...).

```bash
git remote -v                   # xem danh sách remote đang kết nối
git fetch                       # tải thay đổi từ remote về nhưng chưa merge vào local
git pull                        # fetch + merge vào branch hiện tại
git pull --rebase               # fetch + rebase thay vì merge, giữ lịch sử tuyến tính
git push origin <branch>        # push branch lên remote
git push -u origin <branch>     # push và set upstream, lần sau chỉ cần git push
git push --force-with-lease     # force push nhưng an toàn: hủy nếu remote có commit mới chưa lấy về
```

### Merge & Rebase

Tích hợp thay đổi từ branch này sang branch khác. `merge` giữ nguyên lịch sử, `rebase` làm lịch sử thẳng hàng hơn.

```bash
git merge <branch>              # merge branch chỉ định vào branch hiện tại
git merge --no-ff <branch>      # luôn tạo merge commit, dễ truy vết lịch sử hơn
git rebase <branch>             # đặt lại base của branch hiện tại lên đầu branch chỉ định
git rebase -i HEAD~<n>          # interactive rebase: gộp commit (squash), sửa message, xóa commit
```

### Stash

Lưu tạm thay đổi đang dở khi cần chuyển sang việc khác gấp mà chưa muốn commit.

```bash
git stash                       # đẩy thay đổi chưa commit vào stack tạm
git stash pop                   # lấy lại stash gần nhất và xóa khỏi stack
git stash list                  # xem danh sách các stash đang lưu
git stash drop                  # xóa stash gần nhất mà không apply
```

### Hoàn tác

Sửa sai ở các giai đoạn khác nhau. `revert` an toàn hơn `reset` vì không xóa lịch sử.

```bash
git restore <file>              # bỏ thay đổi chưa stage, khôi phục file về trạng thái commit cuối
git restore --staged <file>     # unstage file nhưng giữ nguyên thay đổi trong working dir
git revert <commit>             # tạo commit mới đảo ngược commit chỉ định, an toàn cho nhánh chung
git reset --soft HEAD~1         # bỏ commit cuối, giữ thay đổi ở trạng thái đã stage
git reset --hard HEAD~1         # bỏ commit cuối và xóa luôn toàn bộ thay đổi, không khôi phục được
```
