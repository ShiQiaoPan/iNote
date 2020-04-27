# 无符号函数 sub_xxxx 调试

## 准备工作

* 使用 `class-dump` 导出头文件，查找 `[BrandProfileReporter report13307WithOpType:subOpType:]` 方法地址

  ```objc

  #import <objc/NSObject.h>

  @class BizProfileV2Resp, BrandProfileEnterInfo, CContact;

  @interface BrandProfileReporter : NSObject

  - (void)report13307WithOpType:(unsigned int)arg1 subOpType:(unsigned int)arg2; // IMP=0x000000010125694c

  @end

  ```

## lldb 调试

* lldb 连接设备

  ```txt
  jiaxw@localhost ~ % lldb
  (lldb) platform select remote-ios
    Platform: remote-ios
   Connected: no
    SDK Path: "/Users/jiaxw/Library/Developer/Xcode/iOS DeviceSupport/13.3.1 (17D50) arm64e"
   SDK Roots: [ 0] "/Users/jiaxw/Library/Developer/Xcode/iOS DeviceSupport/13.3.1 (17D50) arm64e"
   SDK Roots: [ 1] "/Users/jiaxw/Library/Developer/Xcode/iOS DeviceSupport/9.2 (13C75)"
   SDK Roots: [ 2] "/Users/jiaxw/Library/Developer/Xcode/iOS DeviceSupport/12.4.2 (16G114)"
   SDK Roots: [ 3] "/Users/jiaxw/Library/Developer/Xcode/iOS DeviceSupport/13.1.1 (17A854)"
   SDK Roots: [ 4] "/Users/jiaxw/Library/Developer/Xcode/iOS DeviceSupport/13.3.1 (17D5026c)"
   SDK Roots: [ 5] "/Users/jiaxw/Library/Developer/Xcode/iOS DeviceSupport/12.4.4 (16G140)"
   SDK Roots: [ 6] "/Users/jiaxw/Library/Developer/Xcode/iOS DeviceSupport/12.2 (16E227)"
   SDK Roots: [ 7] "/Users/jiaxw/Library/Developer/Xcode/iOS DeviceSupport/12.4.1 (16G102)"
   SDK Roots: [ 8] "/Users/jiaxw/Library/Developer/Xcode/iOS DeviceSupport/12.4.3 (16G130)"
   SDK Roots: [ 9] "/Users/jiaxw/Library/Developer/Xcode/iOS DeviceSupport/13.1.3 (17A878) arm64e"
   SDK Roots: [10] "/Users/jiaxw/Library/Developer/Xcode/iOS DeviceSupport/10.0.1 (14A403)"
  (lldb) process connect connect://localhost:8888
  Process 1103 stopped
  * thread #1, queue = 'com.apple.main-thread', stop reason = signal SIGSTOP
      frame #0: 0x000000018c78016c libsystem_kernel.dylib`mach_msg_trap + 8
  libsystem_kernel.dylib`mach_msg_trap:
  ->  0x18c78016c <+8>: ret

  libsystem_kernel.dylib`mach_msg_overwrite_trap:
      0x18c780170 <+0>: mov    x16, #-0x20
      0x18c780174 <+4>: svc    #0x80
      0x18c780178 <+8>: ret
  Target 0: (WeChat) stopped.]
  ```

* `image list` 查看 App 虚拟地址偏移量

  ```txt
  (lldb) image list -o -f | grep WeChat
  [  0] 0x0000000000088000 /var/containers/Bundle/Application/B8B5D334-31EC-4973-AC22-01CD9689FF9D/WeChat.app/WeChat(0x0000000100088000)
  Process 1103 resuming
  ```

* 添加地址断点，`0x000000010125694c` 是符号在可执行文件中偏移地址

  ```txt
  (lldb) br s -a 0x000000010125694c+0x0000000000088000
  Breakpoint 2: where = WeChat`___lldb_unnamed_symbol60465$$WeChat, address = 0x00000001012de94c
  Process 1103 stopped
  * thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 2.1
      frame #0: 0x00000001012de94c WeChat`___lldb_unnamed_symbol60465$$WeChat
  WeChat`___lldb_unnamed_symbol60465$$WeChat:
  ->  0x1012de94c <+0>:  sub    sp, sp, #0x90             ; =0x90
      0x1012de950 <+4>:  stp    x26, x25, [sp, #0x40]
      0x1012de954 <+8>:  stp    x24, x23, [sp, #0x50]
      0x1012de958 <+12>: stp    x22, x21, [sp, #0x60]
  Target 0: (WeChat) stopped.
  ```

* 使用 `di -s <start_address> -e <end_address>` 查看汇编代码

  ```txt
  (lldb) di -s 0x00000001012de94c -e (0x00000001012de94c+0x000000000000013c)
  WeChat`___lldb_unnamed_symbol60465$$WeChat:
  ->  0x1012de94c <+0>:   sub    sp, sp, #0x90             ; =0x90
      0x1012de950 <+4>:   stp    x26, x25, [sp, #0x40]
      0x1012de954 <+8>:   stp    x24, x23, [sp, #0x50]
      0x1012de958 <+12>:  stp    x22, x21, [sp, #0x60]
      0x1012de95c <+16>:  stp    x20, x19, [sp, #0x70]
      0x1012de960 <+20>:  stp    x29, x30, [sp, #0x80]
      0x1012de964 <+24>:  add    x29, sp, #0x80            ; =0x80
      0x1012de968 <+28>:  mov    x22, x0
      0x1012de96c <+32>:  mov    w8, #0x64
      0x1012de970 <+36>:  madd   w25, w2, w8, w3
      0x1012de974 <+40>:  adrp   x8, 29222
      0x1012de978 <+44>:  ldr    x20, [x8, #0x390]
      0x1012de97c <+48>:  ldr    x0, [x0, #0x10]
      0x1012de980 <+52>:  adrp   x8, 28992
      0x1012de984 <+56>:  ldr    x1, [x8, #0xe58]
      0x1012de988 <+60>:  bl     0x106245078               ; symbol stub for: objc_msgSend
      0x1012de98c <+64>:  mov    x29, x29
      0x1012de990 <+68>:  bl     0x1062450e4               ; symbol stub for: objc_retainAutoreleasedReturnValue
      0x1012de994 <+72>:  mov    x19, x0
      0x1012de998 <+76>:  ldr    x0, [x22, #0x18]
      0x1012de99c <+80>:  adrp   x8, 29075
      0x1012de9a0 <+84>:  ldr    x1, [x8, #0x120]
      0x1012de9a4 <+88>:  bl     0x106245078               ; symbol stub for: objc_msgSend
      0x1012de9a8 <+92>:  mov    x29, x29
      0x1012de9ac <+96>:  bl     0x1062450e4               ; symbol stub for: objc_retainAutoreleasedReturnValue
      0x1012de9b0 <+100>: mov    x21, x0
      0x1012de9b4 <+104>: adrp   x8, 28995
      0x1012de9b8 <+108>: ldr    x1, [x8, #0x1d0]
      0x1012de9bc <+112>: bl     0x106245078               ; symbol stub for: objc_msgSend
      0x1012de9c0 <+116>: mov    x29, x29
      0x1012de9c4 <+120>: bl     0x1062450e4               ; symbol stub for: objc_retainAutoreleasedReturnValue
      0x1012de9c8 <+124>: mov    x23, x0
      0x1012de9cc <+128>: adrp   x8, 29225
      0x1012de9d0 <+132>: ldr    x0, [x8, #0x9d8]
      0x1012de9d4 <+136>: adrp   x8, 28995
      0x1012de9d8 <+140>: ldr    x1, [x8, #0x1e8]
      0x1012de9dc <+144>: bl     0x106245078               ; symbol stub for: objc_msgSend
      0x1012de9e0 <+148>: mov    x24, x0
      0x1012de9e4 <+152>: ldr    x0, [x22, #0x10]
      0x1012de9e8 <+156>: adrp   x8, 29072
      0x1012de9ec <+160>: ldr    x1, [x8, #0x620]
      0x1012de9f0 <+164>: bl     0x106245078               ; symbol stub for: objc_msgSend
      0x1012de9f4 <+168>: adrp   x8, 28992
      0x1012de9f8 <+172>: ldr    x1, [x8, #0xe80]
      0x1012de9fc <+176>: stp    x24, x0, [sp, #0x28]
      0x1012dea00 <+180>: orr    w8, wzr, #0x4
      0x1012dea04 <+184>: stp    x8, x23, [sp, #0x18]
      0x1012dea08 <+188>: adrp   x2, 25592
      0x1012dea0c <+192>: add    x2, x2, #0x540            ; =0x540
      0x1012dea10 <+196>: orr    w8, wzr, #0x3
      0x1012dea14 <+200>: str    x19, [sp]
      0x1012dea18 <+204>: stp    x8, x25, [sp, #0x8]
      0x1012dea1c <+208>: mov    x0, x20
      0x1012dea20 <+212>: bl     0x106245078               ; symbol stub for: objc_msgSend
      0x1012dea24 <+216>: mov    x29, x29
      0x1012dea28 <+220>: bl     0x1062450e4               ; symbol stub for: objc_retainAutoreleasedReturnValue
      0x1012dea2c <+224>: mov    x20, x0
      0x1012dea30 <+228>: mov    x0, x23
      0x1012dea34 <+232>: bl     0x1062450a8               ; symbol stub for: objc_release
      0x1012dea38 <+236>: mov    x0, x21
      0x1012dea3c <+240>: bl     0x1062450a8               ; symbol stub for: objc_release
      0x1012dea40 <+244>: mov    x0, x19
      0x1012dea44 <+248>: bl     0x1062450a8               ; symbol stub for: objc_release
      0x1012dea48 <+252>: mov    w0, #0x33fb
      0x1012dea4c <+256>: mov    w2, #0x0
      0x1012dea50 <+260>: mov    w3, #0x0
      0x1012dea54 <+264>: mov    x1, x20
      0x1012dea58 <+268>: bl     0x1000d4724               ; ___lldb_unnamed_symbol837$$WeChat
      0x1012dea5c <+272>: mov    x0, x20
      0x1012dea60 <+276>: ldp    x29, x30, [sp, #0x80]
      0x1012dea64 <+280>: ldp    x20, x19, [sp, #0x70]
      0x1012dea68 <+284>: ldp    x22, x21, [sp, #0x60]
      0x1012dea6c <+288>: ldp    x24, x23, [sp, #0x50]
      0x1012dea70 <+292>: ldp    x26, x25, [sp, #0x40]
      0x1012dea74 <+296>: add    sp, sp, #0x90             ; =0x90
      0x1012dea78 <+300>: b      0x1062450a8               ; symbol stub for: objc_release

    WeChat`___lldb_unnamed_symbol60466$$WeChat:
      0x1012dea7c <+0>:   stp    x20, x19, [sp, #-0x20]!
      0x1012dea80 <+4>:   stp    x29, x30, [sp, #0x10]
      0x1012dea84 <+8>:   add    x29, sp, #0x10            ; =0x10
  ```

* 地址 `0x1000d4724` 为无符号函数内存地址，减去虚拟地址偏移量，即为符号相对于 mach-o 可执行文件偏移地址

  ```txt
  (lldb) p/x 0x1000d4724-0x0000000000088000
  (long) $3 = 0x000000010004c724
  ```

* 添加地址断点

  ```txt
  (lldb) br s -a 0x1000d4724
  Breakpoint 6: where = WeChat`___lldb_unnamed_symbol837$$WeChat, address = 0x00000001000d4724
  Process 1103 stopped
  * thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 6.1
      frame #0: 0x00000001000d4724 WeChat`___lldb_unnamed_symbol837$$WeChat
  WeChat`___lldb_unnamed_symbol837$$WeChat:
  ->  0x1000d4724 <+0>: b      0x1000d4d58               ; ___lldb_unnamed_symbol841$$WeChat

  WeChat`___lldb_unnamed_symbol838$$WeChat:
      0x1000d4728 <+0>: sub    sp, sp, #0x90             ; =0x90
      0x1000d472c <+4>: stp    d9, d8, [sp, #0x30]
      0x1000d4730 <+8>: stp    x26, x25, [sp, #0x40]
  Target 0: (WeChat) stopped.
  ```

* 继续运行，命中断点

  ```txt
  (lldb) c
  Process 1103 resuming
  Process 1103 stopped
  * thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 6.1
      frame #0: 0x00000001000d4724 WeChat`___lldb_unnamed_symbol837$$WeChat
  WeChat`___lldb_unnamed_symbol837$$WeChat:
  ->  0x1000d4724 <+0>: b      0x1000d4d58               ; ___lldb_unnamed_symbol841$$WeChat

  WeChat`___lldb_unnamed_symbol838$$WeChat:
      0x1000d4728 <+0>: sub    sp, sp, #0x90             ; =0x90
      0x1000d472c <+4>: stp    d9, d8, [sp, #0x30]
      0x1000d4730 <+8>: stp    x26, x25, [sp, #0x40]
  Target 0: (WeChat) stopped.
  ```

* 读取寄存器值

  ```txt
  (lldb) re read
  General Purpose Registers:
          x0 = 0x00000000000033fb
          x1 = 0x000000014392b130
          x2 = 0x0000000000000000
          x3 = 0x0000000000000000
          x4 = 0x0000000000000003
          x5 = 0x0000000000000000
          x6 = 0x0000000000000037
          x7 = 0x0000000000000000
          x8 = 0x0000000000000000
          x9 = 0x0000000000000000
         x10 = 0x000000014099aa00
         x11 = 0x000000a1000000ff
         x12 = 0x000000014099b300
         x13 = 0x000001a1b263f751 (0x00000001b263f751) (void *)0x30000001a1b263f7
         x14 = 0x0000000088888900
         x15 = 0x0000000000001b53
         x16 = 0x00000001b263f750  (void *)0x000001a1b263f701
         x17 = 0x000000018d6c95a0  CoreFoundation`-[__NSCFString release]
         x18 = 0x0000000000000000
         x19 = 0x0000000175451bb0
         x20 = 0x000000014392b130
         x21 = 0x000000014390bfd0
         x22 = 0x0000000176c5c200
         x23 = 0x0000000143924620
         x24 = 0x0000000000000000
         x25 = 0x00000000000003e9
         x26 = 0x000000019414ec04  "addButtonWithTitle:"
         x27 = 0x00000001742b6440
         x28 = 0x000000017423bc40
          fp = 0x000000016fd74c50
          lr = 0x00000001012dea5c  WeChat`___lldb_unnamed_symbol60465$$WeChat + 272
          sp = 0x000000016fd74bd0
          pc = 0x00000001000d4724  WeChat`___lldb_unnamed_symbol837$$WeChat
        cpsr = 0x60000000
  ```

* 打印寄存器变量值

  ```txt
  (lldb) po $x0
  13307

  (lldb) po $x1
  gh_880ad183bdca,3,1001,4,gh_880ad183bdca,0,87
  ```