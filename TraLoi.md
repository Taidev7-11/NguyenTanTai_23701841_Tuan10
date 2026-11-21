# Câu trả lời lý thuyết bổ sung (Buổi 10 - BookStore)

### 1. Vì sao khi đổi `numColumns` của `FlatList` phải đổi cả `key`?
`FlatList` tính toán layout lưới cố định ngay từ lần render đầu tiên nên không hỗ trợ thay đổi số cột một cách linh hoạt (on the fly). Việc đổi thuộc tính `key` ép React unmount component cũ và mount lại một component `FlatList` hoàn toàn mới với cấu trúc cột tương ứng; nếu không đổi `key`, ứng dụng sẽ gặp lỗi *"Changing numColumns on the fly is not supported"*.

### 2. Phân biệt dữ liệu lưu ở `useState` và Redux
- **`useState`:** Dùng cho state cục bộ chỉ phục vụ phạm vi giao diện của một màn hình hoặc một component riêng lẻ và không cần lưu trữ hay chia sẻ khi người dùng rời màn hình (ví dụ: từ khóa tìm kiếm `keyword`, chế độ xem lưới `isTile`, lọc `category`).
- **Redux:** Dùng cho dữ liệu toàn cục (global state) cần được chia sẻ, duy trì và đồng bộ tức thời giữa nhiều màn hình khác nhau trong ứng dụng (ví dụ: danh sách sách trong giỏ hàng, số lượng badge hiển thị trên thanh tab bar, danh sách ID sách yêu thích).

### 3. Cơ chế hoạt động của Immer trong `createSlice`
Redux Toolkit tích hợp sẵn thư viện Immer bên trong `createSlice`. Khi reducer được gọi, Immer bọc state hiện tại bằng một proxy gọi là bản nháp (`draft state`), cho phép lập trình viên viết code theo phong cách đột biến trực tiếp (mutating style như `push`, `splice`, gán thuộc tính). Sau khi reducer thực thi xong, Immer tự động ghi nhận các thay đổi trên bản nháp và sinh ra một bản sao state mới hoàn toàn bất biến (immutable), đảm bảo tính toàn vẹn và an toàn cho Redux.

### 4. Lợi ích của `selectIsFavorite(id)` và việc chỉ lưu `id`
- **`selectIsFavorite(id)`:** Là một curried selector (hàm nhận vào `id` và trả về một selector chuẩn `(state) => boolean`). Cách viết này giúp truyền trực tiếp vào hook `useAppSelector` một cách gọn gàng và tối ưu việc re-render của component khi trạng thái yêu thích của cuốn sách đó thay đổi.
- **Chỉ lưu mảng `ids`:** Giúp đảm bảo nguyên tắc nguồn chân lý duy nhất (Single Source of Truth) và tránh trùng lặp dữ liệu trong store. Khi thông tin chi tiết của sách (giá cả, trạng thái kho, đánh giá...) được cập nhật ở danh sách gốc, màn hình Yêu thích sẽ tự động phản ánh đúng dữ liệu mới nhất mà không lo bị lệch dữ liệu.

### 5. Tác dụng của `useCallback` kết hợp với `React.memo`
- `React.memo` bọc quanh component `BookCard` giúp component chỉ re-render khi các props truyền vào bị thay đổi tham chiếu (shallow comparison).
- Nếu không dùng `useCallback` cho callback `onSelect` tại `HomeScreen`, mỗi lần `HomeScreen` re-render (ví dụ khi người dùng nhập từ khóa tìm kiếm), một hàm `onSelect` mới với địa chỉ tham chiếu mới sẽ được tạo ra. Khi đó, `React.memo` sẽ hiểu là props đã thay đổi và buộc toàn bộ danh sách `BookCard` phải render lại, làm mất tác dụng tối ưu hiệu năng của `React.memo`.

### 6. (Bonus) Biến đổi state qua 3 action của `createAsyncThunk`
Khi thực thi một thunk được tạo bởi `createAsyncThunk` (như `fetchBooks`), state sẽ trải qua 3 vòng đời action tương ứng:
- **`pending`:** Được dispatch ngay khi thunk bắt đầu chạy: `state.status = "loading"`, `state.error = null`.
- **`fulfilled`:** Được dispatch khi Promise resolve thành công: `state.status = "succeeded"`, `state.items = action.payload` (gán dữ liệu danh sách sách trả về từ API).
- **`rejected`:** Được dispatch khi có lỗi xảy ra (HTTP lỗi hoặc lỗi mạng): `state.status = "failed"`, `state.error = action.payload ?? "Không tải được dữ liệu"`.
