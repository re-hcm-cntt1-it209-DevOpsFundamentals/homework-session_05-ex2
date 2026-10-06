# Bài 2: Tái cấu trúc lịch sử commit bằng Interactive Rebase

## 1. Mục tiêu

Thực hành sử dụng **Interactive Rebase** để tái cấu trúc lịch sử commit trong Git.

Các yêu cầu:

* Gộp các commit nhỏ thành một commit có ý nghĩa.
* Loại bỏ commit không cần thiết.
* Chỉnh sửa commit message.
* Tạo lịch sử commit sạch và dễ đọc.

---

## 2. Chuẩn bị project

Khởi tạo repository Git:

```bash
git init
```

Tạo file `auth.js` với chức năng đăng nhập:

```javascript
function login(username, password) {
    return username && password;
}
```

Commit đầu tiên:

```bash
git add auth.js
git commit -m "feat: khoi tao module auth"
```

Commit:

```text
de119a5 feat: khoi tao module auth
```

---

## 3. Tạo các commit nhỏ

### Commit 2: `fix typo`

Bổ sung chức năng `logout()`:

```javascript
function login(username, password) {
    return username && password;
}

function logout() {
    console.log("User logged out");
}
```

Commit:

```bash
git add auth.js
git commit -m "fix typo"
```

Commit:

```text
26f8341 fix typo
```

### Commit 3: `adds utility functions`

Bổ sung chức năng `isAuthenticated()`:

```javascript
function login(username, password) {
    return username && password;
}

function logout() {
    console.log("User logged out");
}

function isAuthenticated(user) {
    return user != null;
}
```

Commit:

```bash
git add auth.js
git commit -m "adds utility functions"
```

Commit:

```text
569fa5d adds utility functions
```

### Commit 4: `add temp file for debug`

Tạo file tạm thời để mô phỏng một commit không cần thiết:

```bash
touch temp.txt
echo "temporary debug file" > temp.txt
git add temp.txt
git commit -m "add temp file for debug"
```

Commit:

```text
04f64b5 add temp file for debug
```

---

## 4. Kiểm tra lịch sử trước khi Rebase

Chạy:

```bash
git log --oneline
```

Kết quả:

```text
04f64b5 (HEAD -> master) add temp file for debug
569fa5d adds utility functions
26f8341 fix typo
de119a5 feat: khoi tao module auth
```

Lúc này lịch sử có 4 commit, trong đó có các commit nhỏ và commit `temp.txt` không cần thiết.

---

## 5. Thực hiện Interactive Rebase

Ban đầu thử sử dụng:

```bash
git rebase -i HEAD~4
```

Git trả về lỗi:

```text
fatal: invalid upstream 'HEAD~4'
```

### Nguyên nhân

Repository chỉ có đúng 4 commit.

`HEAD~4` sẽ trỏ đến commit cha của commit đầu tiên, nhưng commit đầu tiên không có commit cha.

Do đó sử dụng:

```bash
git rebase -i --root
```

`--root` cho phép Interactive Rebase bắt đầu từ commit đầu tiên của repository.

---

## 6. Cấu hình Interactive Rebase

Git hiển thị danh sách:

```text
pick de119a5 feat: khoi tao module auth
pick 26f8341 fix typo
pick 569fa5d adds utility functions
pick 04f64b5 add temp file for debug
```

Thay đổi thành:

```text
pick de119a5 feat: khoi tao module auth
squash 26f8341 fix typo
squash 569fa5d adds utility functions
drop 04f64b5 add temp file for debug
```

### Ý nghĩa

| Lệnh     | Ý nghĩa                              |
| -------- | ------------------------------------ |
| `pick`   | Giữ lại commit                       |
| `squash` | Gộp commit vào commit ngay phía trên |
| `drop`   | Loại bỏ hoàn toàn commit             |

Trong bài này:

* `de119a5` được giữ lại bằng `pick`.
* `26f8341` được gộp bằng `squash`.
* `569fa5d` được gộp bằng `squash`.
* `04f64b5` bị loại bỏ bằng `drop`.

### Screenshot Interactive Rebase

Ảnh chụp màn hình Interactive Rebase được thực hiện tại bước này, thể hiện:

```text
pick de119a5 feat: khoi tao module auth
squash 26f8341 fix typo
squash 569fa5d adds utility functions
drop 04f64b5 add temp file for debug
```

> **Chèn ảnh screenshot Interactive Rebase tại đây.**

---

## 7. Chỉnh sửa Commit Message

Sau khi `squash`, Git yêu cầu chỉnh sửa commit message của commit mới.

Commit message cuối cùng:

```text
feat: hoan thien module authentication
```

Git tạo commit mới:

```text
4359fda feat: hoan thien module authentication
```

Commit hash thay đổi là bình thường vì Interactive Rebase đã viết lại lịch sử Git.

---

## 8. Kiểm tra kết quả sau Rebase

### Kiểm tra trạng thái Git

Chạy:

```bash
git status
```

Kết quả:

```text
On branch master
nothing to commit, working tree clean
```

### Kiểm tra lịch sử commit

Chạy:

```bash
git log --oneline
```

Kết quả:

```text
4359fda (HEAD -> master) feat: hoan thien module authentication
```

Chỉ còn **một commit duy nhất**.

### Kiểm tra file

Project chỉ còn:

```text
auth.js
```

File `temp.txt` đã bị loại bỏ.

### Nội dung cuối cùng của `auth.js`

```javascript
function login(username, password) {
    return username && password;
}

function logout() {
    console.log("User logged out");
}

function isAuthenticated(user) {
    return user != null;
}
```

---

## 9. Kết quả

### Trước khi Rebase

```text
04f64b5 add temp file for debug
569fa5d adds utility functions
26f8341 fix typo
de119a5 feat: khoi tao module auth
```

### Sau khi Rebase

```text
4359fda (HEAD -> master) feat: hoan thien module authentication
```

### Kết quả đạt được

* Các commit `fix typo` và `adds utility functions` được gộp thành một commit.
* Commit `add temp file for debug` được loại bỏ.
* File `temp.txt` không còn tồn tại.
* Commit message được đổi thành `feat: hoan thien module authentication`.
* Working tree sạch.
* Lịch sử Git trở nên ngắn gọn và dễ đọc hơn.

---

## 10. Kiến thức rút ra

Interactive Rebase là công cụ dùng để **viết lại lịch sử commit ở local** trước khi chia sẻ lịch sử đó lên remote.

Các lệnh thường sử dụng:

```bash
git rebase -i HEAD~N
```

Hoặc khi muốn rebase từ commit đầu tiên:

```bash
git rebase -i --root
```

Một số thao tác quan trọng:

```text
pick    → giữ commit
reword  → giữ commit nhưng sửa commit message
edit    → dừng lại để chỉnh sửa commit
squash  → gộp commit vào commit phía trên
drop    → xóa commit khỏi lịch sử
```

Trong bài này sử dụng:

```text
pick   → giữ commit đầu tiên
squash → gộp các commit nhỏ
drop   → loại bỏ commit không cần thiết
```

Interactive Rebase làm thay đổi lịch sử commit nên **commit hash có thể thay đổi** sau khi Rebase. Đây là hành vi bình thường.

---

## 11. Cấu trúc thư mục cuối cùng

```text
bai2/
├── .git/
├── README.md
└── auth.js
```

Không còn file:

```text
temp.txt
```

---

## 12. Screenshot

Nếu screenshot được lưu trong thư mục `images` với tên:

```text
images/interactive-rebase.png
```

Có thể chèn vào README bằng:

```markdown
![Interactive Rebase](./images/interactive-rebase.png)
```

Nếu chưa lưu screenshot thành file, có thể giữ nội dung:

> **Chèn ảnh screenshot Interactive Rebase tại đây.**
