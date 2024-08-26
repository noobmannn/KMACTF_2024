# PUSH window, POP nothing

``Author: Lord Of Demons``

![image](https://github.com/user-attachments/assets/47fc8013-0c63-4704-8557-7a0ebc39a687)

Challenge cho một File ``KMACTF.exe`` và yêu cầu người dùng nhập Flag

![image](https://github.com/user-attachments/assets/3d36980e-f4f3-4bca-997b-df5118a15c6c)

Nếu người dùng nhập flag không đủ dài, một MessageBox với nội dung ``Khong du dai`` sẽ in ra, tương tự với việc nhập sai flag thì MessageBox ``wrong`` cũng hiện lên, nhưng vấn đề sẽ xảy ra sau khi tắt MessageBox đi :))) Cái icon như hình dưới sẽ hiện ra và kéo hết mọi tab trong máy vào góc, lúc này bạn không còn cách nào khác ngoài việc phải Restart máy :v 

![image](https://github.com/user-attachments/assets/e2b5f9ec-48e2-4ff6-b534-e8e09d71df89)

### Tổng quan

Mở ``KMACTF.exe`` bằng IDA, ta thấy hàm ``WinMain`` như dưới đây, với hàm ``GenerateEnterFlagBox`` chỉ là định nghĩa và tạo cái MessageBox nhập Flag, vòng while ở dưới nhằm nhận những thao tác từ người dùng

![image](https://github.com/user-attachments/assets/11ce685d-9b7b-4f32-9547-04050501e466)

Tại ``InitWinMain`` có hàm ``loadfunc`` là hàm sẽ xử lý những thao tác nhận được từ người dùng

```c
ATOM __fastcall InitWinMain(HINSTANCE a1)
{
  WNDCLASSEXW v2; // [rsp+20h] [rbp-68h] BYREF

  v2.cbSize = 80;
  v2.style = 3;
  v2.lpfnWndProc = (WNDPROC)loadfunc;
  v2.cbClsExtra = 0;
  v2.cbWndExtra = 0;
  v2.hInstance = a1;
  v2.hIcon = LoadIconW(a1, (LPCWSTR)0x6B);
  v2.hCursor = LoadCursorW(0i64, (LPCWSTR)0x7F00);
  v2.hbrBackground = (HBRUSH)6;
  v2.lpszMenuName = (LPCWSTR)109;
  v2.lpszClassName = &ClassName;
  v2.hIconSm = LoadIconW(v2.hInstance, (LPCWSTR)0x6C);
  return RegisterClassExW(&v2);
}
```

Hàm ``loadfunc`` có cấu trúc như dưới đây

```c
LRESULT __fastcall loadfunc(HWND a1, UINT a2, WPARAM a3, LPARAM a4)
{
  struct _LIST_ENTRY *maybeAddVectorExecptionHandler; // [rsp+40h] [rbp-88h]
  struct tagPAINTSTRUCT Paint; // [rsp+60h] [rbp-68h] BYREF

  switch ( a2 )
  {
    case 2u:
      PostQuitMessage(0);
      ExitProcess(0);
    case 0xFu:
      BeginPaint(a1, &Paint);
      maybeAddVectorExecptionHandler = resAPI(0xFDE7F515, 0xF9D7E6D5);
      ((void (__fastcall *)(__int64, __int64 (__fastcall *)(__int64)))maybeAddVectorExecptionHandler)(1i64, Handle);
      EndPaint(a1, &Paint);
      break;
    case 0x111u:
      switch ( (unsigned __int16)a3 )
      {
        case 'g':
          checkLength(a1);
          break;
        case 'h':
          DialogBoxParamW(hInstance, (LPCWSTR)0x67, a1, DialogFunc, 0i64);
          break;
        case 'i':
          CloseHandle(hObject);
          DestroyWindow(a1);
          break;
        default:
          return DefWindowProcW(a1, a2, a3, a4);
      }
      break;
    default:
      return DefWindowProcW(a1, a2, a3, a4);
  }
  return 0i64;
}
```

Đầu tiên chương trình sẽ chạy vào hàm ``loadfunc``, sau đó nhảy vào case ``0xF`` rồi nhảy vào hàm ``resAPI``, đọc qua hàm này thì nó có cấu trúc giống như các hàm Resolve API thông thường

![image](https://github.com/user-attachments/assets/6389397f-6b5c-4f44-89d5-7688d25a46bf)

Nhưng vấn đề là sau khi Resolve API thì chương trình chạy đến hàm ``loadresourcesss``

```c
__int64 loadresourcesss()
{
  DWORD TempPathW; // [rsp+50h] [rbp-4F8h]
  DWORD nNumberOfBytesToWrite; // [rsp+54h] [rbp-4F4h]
  unsigned int v3; // [rsp+58h] [rbp-4F0h]
  HRSRC hResInfo; // [rsp+60h] [rbp-4E8h]
  HANDLE hFile; // [rsp+68h] [rbp-4E0h]
  HGLOBAL hResData; // [rsp+70h] [rbp-4D8h]
  LPCVOID lpBuffer; // [rsp+78h] [rbp-4D0h]
  DWORD NumberOfBytesWritten; // [rsp+80h] [rbp-4C8h] BYREF
  struct _PROCESS_INFORMATION ProcessInformation; // [rsp+88h] [rbp-4C0h] BYREF
  struct _STARTUPINFOW StartupInfo; // [rsp+A0h] [rbp-4A8h] BYREF
  WCHAR FileName[264]; // [rsp+110h] [rbp-438h] BYREF
  WCHAR Buffer[264]; // [rsp+320h] [rbp-228h] BYREF

  hResInfo = FindResourceW(0i64, (LPCWSTR)0x82, L"BIN");
  if ( !hResInfo )
    return 0i64;
  hResData = LoadResource(0i64, hResInfo);
  if ( !hResData )
    return 0i64;
  nNumberOfBytesToWrite = SizeofResource(0i64, hResInfo);
  if ( !nNumberOfBytesToWrite )
    return 0i64;
  lpBuffer = LockResource(hResData);
  if ( !lpBuffer )
    return 0i64;
  TempPathW = GetTempPathW(0x104u, Buffer);
  if ( !TempPathW || TempPathW > 0x104 )
    return 0i64;
  wsprintfW(FileName, L"%s%s", Buffer, L"Windows Update Checker 2.exe");
  hFile = CreateFileW(FileName, 0x40000000u, 0, 0i64, 2u, 0x80u, 0i64);
  if ( hFile == (HANDLE)-1i64 )
    return 0i64;
  NumberOfBytesWritten = 0;
  v3 = WriteFile(hFile, lpBuffer, nNumberOfBytesToWrite, &NumberOfBytesWritten, 0i64);
  CloseHandle(hFile);
  memset(&StartupInfo, 0, sizeof(StartupInfo));
  StartupInfo.cb = 104;
  memset(&ProcessInformation, 0, sizeof(ProcessInformation));
  CreateProcessW(FileName, 0i64, 0i64, 0i64, 0, 0x10u, 0i64, 0i64, &StartupInfo, &ProcessInformation);
  return v3;
```

Tại đây chương trình load một số byte từ phần resources ``BIN`` của file exe, thể hiện qua các API liên quan đến resource được gọi (``FindResourceW``, ``LoadResource``, ``SizeofResource`` và ``LockResource``), sau đó gọi hàm ``GetTempPathW`` để lấy đường dẫn Folder Temp trong máy và nối với chuỗi ``Windows Update Checker 2.exe``, sau đó gọi hàm ``WriteFile`` để viết resource vào File vừa được tạo. Cuối cùng là gọi hàm ``CreateProcessW`` để chạy một Process mới với file vừa được tạo trên.

Quay trở lại ``loadfunc``, chương trình sau khi bắt người dùng nhập flag sẽ chạy đến case ``g`` trong case ``0x111`` rồi chạy vào hàm ``checkLength``

```c
void __fastcall __noreturn checkLength(HWND a1)
{
  HWND DlgItem; // rax
  unsigned __int64 v2; // [rsp+20h] [rbp-248h]
  _BYTE *v3; // [rsp+28h] [rbp-240h]
  __int16 String[256]; // [rsp+50h] [rbp-218h] BYREF

  DlgItem = GetDlgItem(a1, 102);
  GetWindowTextW(DlgItem, (LPWSTR)String, 256);
  v2 = -1i64;
  do
    ++v2;
  while ( String[v2] );
  if ( v2 < 0x2E )
  {
    MessageBoxW(a1, L"Khong du dai", L"Khong du dai", 0);
    v3 = operator new(1ui64);
    *v3 = 85;
    connectpipe(v3, 1u);
    ExitProcess(0);
  }
  MovePipe((const WCHAR *)String);
  NextHandl();
}
```

Ở đây có thể dễ thấy chương trình gọi ``GetWindowTextW`` để lấy nội dung được người dùng nhập vào, sau đó tính độ dài chuỗi rồi check với giá trị ``0x2E``, nếu bé hơn thì tạo rồi in MessageBox với nội dung ``Khong du dai`` và gọi hàm ``connectpipe`` với tham số là ``0x55``

Nếu độ dài đúng, chương trình chạy vào hàm ``MovePipe``, về cơ bản hàm này tạo ra một mảng gồm số 1 ở đầu và flag người dùng nhập vào ở sau rồi truyền mảng trên vào hàm ``connectpipe``

![image](https://github.com/user-attachments/assets/2d61a281-52bc-485d-a426-7d4e0977b52b)

Sau đó chương trình tiếp tục chạy đến hàm ``NextHandl``

![image](https://github.com/user-attachments/assets/6b6fd9d0-07ec-4d02-bdba-ab14a3279820)

Hàm này gọi ``connectpipe`` với tham số truyền vào là 8, sau đó thực hiện swicth case để vào các hàm gây lỗi sau:
- ``bug1`` : hàm này gây lỗi ``STATUS_ILLEGAL_INSTRUCTION`` với mã lỗi ``0xC000001D``
- ``debugbreak``: hàm này gây lỗi ``STATUS_BREAKPOINT`` với mã lỗi ``0x80000003``
- ``unkdat``: hàm này gây lỗi ``STATUS_ACCESS_VIOLATION`` với mã lỗi ``0xC0000005``
- ``PreviegledExecpt``: hàm này gây lỗi ``STATUS_PRIVILEGED_INSTRUCTION`` với mã lỗi ``0xC0000096``
- ``dividezeroExecpt``: hàm này gây lỗi ``STATUS_INTEGER_DIVIDE_BY_ZERO`` với mã lỗi ``0xC0000094``

Nhìn lại hàm ``loadfunc``, sau khi phân tích kĩ thì có thể nhận thấy API được resolve từ ``resapi`` khá giống với ``AddVectoredExceptionHandler``, cộng với việc các lỗi được liệt kê ở trong các hàm bên trên giống như tác giả cố tình code như vậy thì có thể nhận thấy đây là ý đồ của tác giả để điều hướng chương trình vào hàm ``Handle``

![image](https://github.com/user-attachments/assets/3bcaf99d-314b-446b-9dcb-7f025945b816)

Phân tích hàm ``Handle``, hàm này truyền tạo mảng gồm giá trị ``5`` và mã lỗi sau đó truyền mảng trên vào ``connectpipe``

```
__int64 __fastcall Handle(_EXCEPTION_POINTERS *a1)
{
  _BYTE *v2; // [rsp+30h] [rbp-58h]
  _EXCEPTION_POINTERS v3; // [rsp+40h] [rbp-48h]

  v3 = *a1;
  v2 = operator new(0x569ui64);
  *v2 = 5;
  qmemcpy(v2 + 1, v3.ExceptionRecord, 0x98ui64);
  qmemcpy(v2 + 153, v3.ContextRecord, 0x4D0ui64);
  connectpipe(v2, 0x569u);
  qmemcpy(v3.ExceptionRecord, v2 + 1, sizeof(EXCEPTION_RECORD));
  qmemcpy(v3.ContextRecord, v2 + 153, sizeof(struct _CONTEXT));
  return 0xFFFFFFFFi64;
}
```
