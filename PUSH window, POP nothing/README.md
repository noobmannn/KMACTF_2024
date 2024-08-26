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

```C
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

```C
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

```C
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

```C
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

### Pipe

Có thể thấy hàm ``connectpipe`` bị gọi khá nhiều lần với nhiều thể loại tham số truyền vào khác nhau, xem qua hàm này thì hàm dùng ``CreateFileW`` để tạo kết nối đến ``\\\\.\\pipe\\KMACTF``, sau đó đó dùng ``WriteFile`` để truyền data đến tiến trình khác thông qua pipe , rồi lại dùng ``ReadFile`` để đọc thông tin được nhận về từ tiến trình kia cũng thông qua pipe

```C
BOOL __fastcall connectpipe(void *a1, DWORD a2)
{
  DWORD NumberOfBytesWritten; // [rsp+40h] [rbp-18h] BYREF
  DWORD NumberOfBytesRead; // [rsp+44h] [rbp-14h] BYREF

  while ( 1 )
  {
    hObject = CreateFileW(L"\\\\.\\pipe\\KMACTF", 0xC0000000, 0, 0i64, 3u, 0, 0i64);
    if ( hObject != (HANDLE)-1i64 )
      break;
    Sleep(0x64u);
  }
  WriteFile(hObject, a1, a2, &NumberOfBytesWritten, 0i64);
  ReadFile(hObject, a1, a2, &NumberOfBytesRead, 0i64);
  return CloseHandle(hObject);
}
```

Quay lại hàm ``loadresource``, hàm này tạo một file có tên ``Windows Update Checker 2.exe`` trong thư mục ``Temp`` sau đó chạy tiến trình bằng file trên. Bây giờ mình sẽ vào thư mục ``Temp`` để mở file ``Windows Update Checker 2.exe`` bằng IDA

Ở file này, chương trình gọi ``WinMain`` -> ``sub_7FF63B351560`` như dưới đây

![image](https://github.com/user-attachments/assets/8f014da9-39d3-4fc4-8284-ffbac91fd01f)

Chương trình gọi ``CreateNamedPipe`` để tạo pipe ``\\\\.\\pipe\\KMACTF`` sau đó gọi ``ConnectNamedPipe`` để kết nối đến pipe đó. Hàm ``ReadFile`` được gọi để nhận data truyền đến từ tiến trình bên kia, như đã phân tích ở các hàm trên, data truyền về sẽ luôn có dạng ``numer + data``, với number được truyền vào trong switch-case

#### Case 0x55

Case này được gọi tới khi người dùng nhập flag không đủ dài

```C
case 0x55:
          anhTaiTrollerdotExe();
          ExitProcess(0);
```

Vào hàm ``anhTaiTrollerdotExe`` ->``sub_7FF63B351330``, hàm này cũng load resource và ghi vào file ``Windows Update Checker.exe`` trong thư mục ``Temp`` rồi dùng ``CreateProcessW`` để chạy tiến trình trên, sau khi thủ dump file ra và phân tích thì có vẻ đây chính là thử gây ra hiện tượng ảnh anh Tài chạy lung tung trên màn hình :))))

#### Case 0x1

```C
case 1:
          memset(flag, 0, sizeof(flag));
          v10 = -1i64;
          do
            ++v10;
          while ( *((_BYTE *)lpBuffer + v10 + 1) );
          qmemcpy(flag, (char *)lpBuffer + 1, v10);
          memset((void *)flag_base64, 0, 0x100ui64);
          v11 = -1i64;
          do
            ++v11;
          while ( flag[v11] );
          flag_base64 = (__int64)base64encode((__int64)flag, v11);
          v12 = -1i64;
          do
            ++v12;
          while ( *(_BYTE *)(flag_base64 + v12) );
          len_base64 = v12;
          WriteFile(hNamedPipe, lpBuffer, 0x600u, &NumberOfBytesWritten, 0i64);
          break;
```

Case này xử lý input người dùng nhập vào, bằng cách mã hoá thành một chuỗi base64 và lưu vào biến ``flag_base64``, data được gửi đi cho chương trình khác là kí tự đầu tiên của chuỗi Base64 bị mã hoá

#### Case 0x5

```C
case 5:
          v3 = *(_DWORD *)((char *)lpBuffer + 1);
          if ( errorcode[idx] != v3 )
            check = 0;
          cur_chr = *(_BYTE *)(flag_base64 + idx);
          switch ( v3 )
          {
            case 0x80000003:
              errorcode_cpy[idx] = 0x80000003;
              for ( i = 0; i < 10; ++i )
                *(_BYTE *)(flag_base64 + idx) = 7 * (i ^ cur_chr) + ((i + 51) ^ (*(_BYTE *)(flag_base64 + idx) + 69));
              ++idx;
              ++*(_QWORD *)((char *)lpBuffer + 401);
              break;
            case 0xC0000005:
              errorcode_cpy[idx] = 0xC0000005;
              for ( j = 0; j < 10; ++j )
                *(_BYTE *)(flag_base64 + idx) = (*(_BYTE *)(flag_base64 + idx) + j + 85) ^ 7;
              ++idx;
              *(_QWORD *)((char *)lpBuffer + 401) += 7i64;
              break;
            case 0xC000001D:
              errorcode_cpy[idx] = 0xC000001D;
              for ( k = 0; k < 10; ++k )
                *(_BYTE *)(flag_base64 + idx) = (*(char *)(flag_base64 + idx) << (k % 3)) & 0x4F ^ (91 * ((k + cur_chr) ^ *(_BYTE *)(flag_base64 + idx))
                                                                                                  + k
                                                                                                  + (cur_chr >> (((k >> 31) ^ k & 1) - (k >> 31))));
              ++idx;
              *(_QWORD *)((char *)lpBuffer + 401) += 2i64;
              break;
            case 0xC0000094:
              errorcode_cpy[idx] = 0xC0000094;
              for ( m = 0; m < 10; ++m )
                *(_BYTE *)(flag_base64 + idx) = (m ^ cur_chr)
                                              + 93
                                              * ((m + cur_chr) ^ (3 * cur_chr + m + *(_BYTE *)(flag_base64 + idx) + 4 * m));
              ++idx;
              *(_QWORD *)((char *)lpBuffer + 401) += 3i64;
              break;
            case 0xC0000096:
              errorcode_cpy[idx] = 0xC0000096;
              for ( n = 0; n < 10; ++n )
                *(_BYTE *)(flag_base64 + idx) = (77
                                               * ((7 * n) ^ (*(char *)(flag_base64 + idx) + (cur_chr << (n % 3)) + 0x2D))
                                               + n
                                               + cur_chr)
                                              % 0xFF;
              ++idx;
              *(_QWORD *)((char *)lpBuffer + 401) += 3i64;
              break;
          }
          WriteFile(hNamedPipe, lpBuffer, 0x600u, &NumberOfBytesWritten, 0i64);
          break;
```

Case này nhận kí tự cần mã hoá và mã lỗi, đầu tiên case này sẽ check mã lỗi với giá trị tại vị trí thứ ``idx`` trong mảng mã lỗi ``errorcode``, nếu sai thì biến check bị gán thành 0, sau khi kiểm tra mã lỗi thì chương trình dựa vào mã lỗi để nhảy đến case mã hoá tương ứng, mã hoá xong thì tăng idx lên, data được gửi đi cho chương trình khác là kí tự tiếp theo của chuỗi Base64 bị mã hoá.

Quay lại file ``KMACTF.exe``, có thể thấy với mỗi kí tự thì hàm gây lỗi được gọi vào sẽ khác nhau, từ đây ta có thể suy ra được với những kí tự nào sẽ nhảy đến hàm gây mã lỗi nào -> nhảy đến case mã hoá nào

![image](https://github.com/user-attachments/assets/90184957-4188-4b2d-ad43-8427b5129670)


#### Case 0x8:

```C
if ( idx >= len_base64 )
          {
            v1 = 1;
            for ( ii = 0; ; ++ii )
            {
              if ( ii >= 64 )
                goto LABEL_52;
              if ( *(char *)(flag_base64 + ii) != checker[ii] )
                break;
            }
            v1 = 0;
LABEL_52:
            *(_BYTE *)lpBuffer = -52;
            WriteFile(hNamedPipe, lpBuffer, 1u, &NumberOfBytesWritten, 0i64);
            check = 1;
            idx = 0;
            if ( v1 )
            {
              MessageBoxW(0i64, L"Correct", L"Correct", 0x40000u);
            }
            else
            {
              MessageBoxW(0i64, L"Wrong", L"Wrong", 0x40000u);
              anhTaiTrollerdotExe();
            }
          }
          else
          {
            *(_BYTE *)lpBuffer = *(_BYTE *)(flag_base64 + idx);
            if ( !check )
            {
              *(_BYTE *)lpBuffer = -52;
              WriteFile(hNamedPipe, lpBuffer, 1u, &NumberOfBytesWritten, 0i64);
              idx = 0;
              check = 1;
              CloseHandle(hNamedPipe);
              MessageBoxW(0i64, L"Wrong", L"Wrong", 0x40000u);
              anhTaiTrollerdotExe();
            }
            WriteFile(hNamedPipe, lpBuffer, 1u, &NumberOfBytesWritten, 0i64);
          }
          break;
```

Case này sẽ check nếu ``idx >= len_base64``, tức là nếu đã mã hoá hết toàn bộ base64 thì sẽ vào vòng lặp để kiểm tra với mảng ``checker`` nếu đung hết thì nhảy ra MessageBox ``correct``, sai thì nhảy ra MessageBox ``wrong``; còn nếu chưa mã hoá hết thì kiểm tra lại biến ``check`` để xem ``errorcode`` được gọi đúng thứ tự không, nếu không thì cũng nhảy ra MessageBox ``wrong`` 

Tổng kết lại, chương trình thực hiện việc mã hoá Flag dựa vào việc truyền data giữa hai tiến trình bằng pipe, flag được nhập vào phải dài ``0x2E``, sau đó flag được mã hoá base64, sau đó chương trình sẽ check từng kí tự một trong chuỗi base64 thu được, dựa vào cơ chế hàm gây lỗi như trên để xác định hàm mã hoá cụ thể cho từng kí tự khác nhau. Mã hoá xong thì gọi đến Case 8 để kiểm tra

### Reverse

Dựa vào những thông tin trên, mình sẽ viết một script để tìm được chuỗi base64 sau đó decode chuỗi base64 đó để lấy flag

```C
#include <stdio.h>
#include <string.h>
#include <windows.h>

BYTE checker[65] = {
    0x72, 0xBB, 0xB2, 0xCD, 0x58, 0xB2, 0x81, 0x0E, 0xA4, 0xB1,
    0xED, 0xDB, 0x84, 0xB2, 0xC0, 0xAA, 0x60, 0xD0, 0xE8, 0xE8,
    0xB0, 0x12, 0x81, 0x1E, 0xED, 0xD0, 0xF3, 0x05, 0xB0, 0xB1,
    0x04, 0x04, 0x7D, 0xF3, 0xC0, 0xE8, 0xED, 0x12, 0xF3, 0xC2,
    0x7D, 0x0E, 0x0E, 0x0E, 0x7D, 0x04, 0xC0, 0xBB, 0xED, 0xB1,
    0x81, 0xED, 0xA4, 0xCF, 0xC0, 0x68, 0x84, 0xD0, 0xE2, 0x1B,
    0xC2, 0x58, 0x30, 0x30, 0x00
};

int errorcode[] = {0xC0000094, 0x0C0000005, 0x0C0000096, 0x0C0000005, 0x0C0000094, 0x0C0000096, 0x0C000001D, 0x0C0000094, 0x0C0000094,
0x0C000001D, 0x0C0000094, 0x0C000001D, 0x0C0000096, 0x0C0000096, 0x0C0000094, 0x80000003, 0x0C0000094, 0x0C0000096, 0x0C0000096, 0x0C0000096,
0x0C000001D, 0x0C0000094, 0x0C000001D, 0x80000003, 0x0C0000005, 0x0C0000096, 0x0C0000094, 0x0C0000005, 0x0C000001D, 0x0C000001D,
0x80000003, 0x0C0000005, 0x0C000001D, 0x0C0000094, 0x0C0000094, 0x0C0000096, 0x0C0000005, 0x0C0000094, 0x0C0000094, 0x0C0000096,
0x0C000001D, 0x0C0000094, 0x0C000001D, 0x0C000001D, 0x0C000001D, 0x80000003, 0x0C0000094, 0x0C0000005, 0x0C0000005, 0x0C000001D, 0x0C000001D,
0x0C0000094, 0x0C0000094, 0x0C0000005, 0x0C0000094, 0x0C000001D, 0x0C0000096, 0x0C0000096, 0x0C0000005, 0x80000003, 0x0C0000096,
0x0C0000094, 0x0C0000096, 0x0C0000096};

BYTE is0xC000001D[13] = {'+', '2', 'H', 'R', 'X', 'Z', 'a', 'l', 'm', 'o', 's', 'v', 'x'};
BYTE is0x80000003[13] = {'/', 'C', 'D', 'F', 'N', 'P', 'c', 'j', 'k', 'p', 'q', 't', 'w'};
BYTE is0xC0000005[13] = {'0', '6', '7', 'A', 'B', 'K', 'L', 'O', 'T', 'b', 'g', 'y', 'z'};
BYTE is0xc0000096[13] = {'1', '4', '5', '8', '=', 'I', 'J', 'U', 'W', 'd', 'f', 'r', 'u'};
BYTE is0xc0000094[13] = {'3', '9', 'E', 'G', 'M', 'Q', 'S', 'V', 'Y', 'e', 'h', 'i', 'n'};

BYTE case0x80000003(BYTE chr){
    BYTE v0 = chr;
    for (int i = 0; i < 10; i++){
        chr = 7 * (i ^ v0) + ((i + 51) ^ (chr + 69));
    }
    return chr;
}

BYTE case0xC0000005(BYTE chr){
    BYTE v0 = chr;
    for (int j = 0; j < 10; j++){
        chr = (chr + j + 85) ^ 7;
    }
    return chr;
}

BYTE case0xC000001D(BYTE chr){
    BYTE v0 = chr;
    for (int k = 0; k < 10; k++){
        chr = ((char)chr << (k % 3)) & 0x4F ^ (91 * ((k + v0) ^ chr) + k + (v0 >> (((k >> 31) ^ k & 1) - (k >> 31))));
    }
    return chr;
}

BYTE case0xC0000094(BYTE chr){
    BYTE v0 = chr;
    for (int m = 0; m < 10; m++){
        chr = (m ^ v0) + 93 * ((m + v0) ^ (3 * v0 + m + chr + 4 * m));
    }
    return chr;
}

BYTE case0xC0000096(BYTE chr){
    BYTE v0 = chr;
    for (int n = 0; n < 10; n++){
        chr = (77 * ((7 * n) ^ ((char)chr + (v0 << (n % 3)) + 45)) + n + v0) % 255;
    }
    return chr;
}

int main(){
    for (int i = 0; i < 64; i++) {
        switch(errorcode[i]){
            case 0x80000003:
                for (int j = 0; j < strlen(is0x80000003); j++) {
                    if (case0x80000003(is0x80000003[j]) == checker[i]) {
                        printf("%c", is0x80000003[j]);
                        break;
                    }
                }
                break;
            case 0xC0000005:
                for (int j = 0; j < strlen(is0xC0000005); j++) {
                    if (case0xC0000005(is0xC0000005[j]) == checker[i]) {
                        printf("%c", is0xC0000005[j]);
                        break;
                    }
                }
                break;
            case 0xC000001D:
                for (int j = 0; j < strlen(is0xC000001D); j++) {
                    if (case0xC000001D(is0xC000001D[j]) == checker[i]) {
                        printf("%c", is0xC000001D[j]);
                        break;
                    }
                }
                break;
            case 0xC0000094:
                for (int j = 0; j < strlen(is0xc0000094); j++) {
                    if (case0xC0000094(is0xc0000094[j]) == checker[i]) {
                        printf("%c", is0xc0000094[j]);
                        break;
                    }
                }
                break;
            case 0xC0000096:
                for (int j = 0; j < strlen(is0xc0000096); j++) {
                    if (case0xC0000096(is0xc0000096[j]) == checker[i]) {
                        printf("%c", is0xc0000096[j]);
                        break;
                    }
                }
                break;
        }
    }
    printf("\n");
    return 0;
}
```

Kết quả script C trên là chuỗi ``S01BQ1RGe2hvd19tYW55X3RpbWVzX2FyZV95b3VfZGllZF90b2RheT9odWg/fQ==``, ném lên Cyberchef và lấy Flag của challenge

![image](https://github.com/user-attachments/assets/8fbee736-78fd-4ddb-8d0b-8f804accd056)

# Flag

``KMACTF{how_many_times_are_you_died_today?huh?}``

# Note

Script giải bằng Python

```python
import base64

checker = [0x72, 0xBB, 0xB2, 0xCD, 0x58, 0xB2, 0x81, 0x0E, 0xA4, 0xB1, 
  0xED, 0xDB, 0x84, 0xB2, 0xC0, 0xAA, 0x60, 0xD0, 0xE8, 0xE8, 
  0xB0, 0x12, 0x81, 0x1E, 0xED, 0xD0, 0xF3, 0x05, 0xB0, 0xB1, 
  0x04, 0x04, 0x7D, 0xF3, 0xC0, 0xE8, 0xED, 0x12, 0xF3, 0xC2, 
  0x7D, 0x0E, 0x0E, 0x0E, 0x7D, 0x04, 0xC0, 0xBB, 0xED, 0xB1, 
  0x81, 0xED, 0xA4, 0xCF, 0xC0, 0x68, 0x84, 0xD0, 0xE2, 0x1B, 
  0xC2, 0x58, 0x30, 0x30, 0x00]

is0xC000001D = ['+', '2', 'H', 'R', 'X', 'Z', 'a', 'l', 'm', 'o', 's', 'v', 'x']
is0x80000003 = ['/', 'C', 'D', 'F', 'N', 'P', 'c', 'j', 'k', 'p', 'q', 't', 'w']
is0xC0000005 = ['0', '6', '7', 'A', 'B', 'K', 'L', 'O', 'T', 'b', 'g', 'y', 'z']
is0xc0000096 = ['1', '4', '5', '8', '=', 'I', 'J', 'U', 'W', 'd', 'f', 'r', 'u']
is0xc0000094 = ['3', '9', 'E', 'G', 'M', 'Q', 'S', 'V', 'Y', 'e', 'h', 'i', 'n']

errorcode = [0xC0000094, 0x0C0000005, 0x0C0000096, 0x0C0000005, 0x0C0000094, 0x0C0000096, 0x0C000001D, 0x0C0000094, 0x0C0000094,
0x0C000001D, 0x0C0000094, 0x0C000001D, 0x0C0000096, 0x0C0000096, 0x0C0000094, 0x80000003, 0x0C0000094, 0x0C0000096, 0x0C0000096, 0x0C0000096,
0x0C000001D, 0x0C0000094, 0x0C000001D, 0x80000003, 0x0C0000005, 0x0C0000096, 0x0C0000094, 0x0C0000005, 0x0C000001D, 0x0C000001D,
0x80000003, 0x0C0000005, 0x0C000001D, 0x0C0000094, 0x0C0000094, 0x0C0000096, 0x0C0000005, 0x0C0000094, 0x0C0000094, 0x0C0000096,
0x0C000001D, 0x0C0000094, 0x0C000001D, 0x0C000001D, 0x0C000001D, 0x80000003, 0x0C0000094, 0x0C0000005, 0x0C0000005, 0x0C000001D, 0x0C000001D,
0x0C0000094, 0x0C0000094, 0x0C0000005, 0x0C0000094, 0x0C000001D, 0x0C0000096, 0x0C0000096, 0x0C0000005, 0x80000003, 0x0C0000096,
0x0C0000094, 0x0C0000096, 0x0C0000096]

ciph = ''

def case0x0C0000094(char):
    v0 = char
    for i in range(10):
        char = ((i ^ v0) + ((93 * ((i + v0) ^ ((3 * v0 + i + char + 4 * i) & 0xFF))) & 0xFF)) & 0xFF 
    return char

def case0x0C0000096(char):
    v0 = char
    for n in range(10):
        if char & 0x80:
            char |= 0xFFFFFF00
        char = (((77 * ((7 * n) ^ ((char + ((v0 << (n % 3))) + 45) & 0xFFFFFFFF))) + n + v0) % 255) & 0xFF
    return char

def case0xC000001D(char):
    v0 = char
    for k in range(10):
        if char & 0x80:
            char |= 0xFFFFFF00
        char = ((char << (k % 3)) & 0xFFFFFFFF) & 0x4F ^ ((((91  * ((k + v0) ^ char)) & 0xFF) + k + (v0 >> (((k >> 31) ^ k & 1) - (k >> 31)))) & 0xFF)
    return char

def case0xC0000005(char):
    v0 = char
    for j in range(10):
        char = ((char + j + 85) & 0xFF) ^ 7
    return char

def case0x80000003(char):
    v0 = char
    for i in range(10):
        char = (((7 * (i ^ v0)) & 0xFF) + ((i + 51) ^ (char + 69))) & 0xFF
    return char

for i in range(64):
    match errorcode[i]:
        case 0x0C000001D:
            for c in is0xC000001D:
                if case0xC000001D(ord(c)) == checker[i]:
                    ciph += c
        case 0x80000003:
            for c in is0x80000003:
                if case0x80000003(ord(c)) == checker[i]:
                    ciph += c
        case 0x0C0000005:
            for c in is0xC0000005:
                if case0xC0000005(ord(c)) == checker[i]:
                    ciph += c
        case 0x0C0000094:
            for c in is0xc0000094:
                if case0x0C0000094(ord(c)) == checker[i]:
                    ciph += c 
        case 0x0C0000096:
            for c in is0xc0000096:
                if case0x0C0000096(ord(c)) == checker[i]:
                    ciph += c

print(base64.b64decode(ciph.encode()))
```
