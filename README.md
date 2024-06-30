# WASI Random

[WebAssembly系统接口(WebAssembly System Interface)](https://github.com/WebAssembly/WASI)API提案。

### 当前阶段（Current Phase）

WASI-random目前处于[第3阶段(Phase 3)][Phase 3]。

[Phase 3]: https://github.com/WebAssembly/WASI/blob/main/Proposals.md#phase-3---implementation-phase-cg--wg

### 拥护者（Champions）

- Dan Gohman

### 可移植性标准（Portability Criteria）


WASI random必须有至少可以在Windows、macOS和Linux上通过测试套件(testsuite)的主机实现。

WASI random必须至少有两个完整独立的实现。

## Table of Contents

- [介绍（Introduction）](#介绍introduction)
- [目标（Goals）](#目标goals)
- [非目标（Non-goals）](#非目标non-goals)
- [API详解（API walk-through）](#API详解api-walk-through)
  - [Use case 1](#use-case-1)
  - [Use case 2](#use-case-2)
- [详细设计讨论（Detailed design discussion）](#详细设计讨论detailed-design-discussion)
  - [[Tricky design choice 1]](#tricky-design-choice-1)
  - [[Tricky design choice 2]](#tricky-design-choice-2)
- [考虑替代方案（Considered alternatives）](#考虑替代方案considered-alternatives)
  - [[Alternative 1]](#alternative-1)
  - [[Alternative 2]](#alternative-2)
- [项目相关方利益 & 反馈（Stakeholder Interest & Feedback）](#项目相关方利益--反馈stakeholder-interest--feedback)
- [参考文献 & 致谢（References & acknowledgements）](#参考文献--致谢references--acknowledgements)

### 介绍（Introduction）

WASI Random 是用于获取伪随机数据(pseudo-random data)的WASI API。

### 目标（Goals）

WASI Random的主要目标是：
 - 允许用户使用WASI程序获取适用于密码学(cryptography)的高质量的低级随机数据(low-level random data)。
 - 允许源语言在支持的宿主环境中启用其哈希映射(hash-maps)中的DoS防护。

### 非目标（Non-goals）

WASI Random不旨在允许程序处理错误或查询可用性。
它总是成功的（尽管在randomness不可用的平台上，程序可能无法实例化或陷入异常状态）。

WASI Random不旨在成为完整的DRBG API（确定性随机比特生成器API）。
此类API可以在WASI中考虑，但应该作为一个单独的提案。

并且，WASI Random不包括用于将熵反馈(entropy back)到系统的功能。
预期应用程序观察到的大多数熵(entropy)也应该被宿主实现所观察到，因此几乎没有必要将其反馈回去。
此类API可能还有其他用途，但他们可以在单独的提案中讨论。

WASI Random没有异步API。
其实现预计使用CSPRNG(密码安全伪随机数生成器)，预期其具备充足的种子(be sufficiently seeded)。

WASI Random没有显式的域分离(domain separation)或个性化消息(personalization messages)的功能。
如果需要此类功能，将它们定义为自定义部分而不是程序数据是合理的，这样就可以轻松地将它们从模块缓存中排除，也可能从代码签名中排除。
这作为单独的提案是合理的。

WASI Random不直接提供“entropy API(熵API)”或“true random(真随机)”API。
目前预期的用例需要CSPRNG API。

WASI Random不公开熵估算(entropy estimation)。它预计总是有足够的熵来播种CSPRNG(have sufficient entropy to seed a CSPRNG)。

WASI Random不提供任何将随机数据(random data)替换为确定性数据(deterministic data)的功能。
它旨在用于确定会打断应用程序假设的用例。实现可能具有使此API具有确定性的调试功能，但这些功能仅应用于调试，而不应用于生产。

### API详解（API walk-through）

#### 主要API：获取加密安全的伪随机字节（Main API: getting cryptographically-secure pseudo-random bytes）

返回加密安全的伪随机字节列表：

```rust
    let len: u32 = your_own_code_to_decide_how_many_bytes_you_want();

    let bytes: Vec<u8> = get_random_bytes(len);
```

#### 主要API：更快地获取加密安全的伪随机字节（Main API: getting cryptographically-secure pseudo-random bytes faster）

有时候`list<u8>`的绑定可能会产产生一些开销，因此可以使用另一个函数返回相同的数据，但为`u64`：

```rust
    let data: u64 = get_random_u64();
```

#### 不安全的API：哈希映射DoS防护（Insecure API: Hash-map DoS protection）

返回一对可用于初始化哈希实现的u64：

```rust
    let init: (u64, u64) = insecure_random();

    let combined: u128 = init.0 as u128 | (init.1 as u128 << 64);

    your_own_code_to_initialize_hash_map(combined);
```

### 详细设计讨论（Detailed design discussion）

### 如果系统在早期启动时缺乏足够的熵会怎样？（What if the system lacks sufficient entropy during early boot?）

随机性API可能会失败，或者可能是“非阻塞”的并返回不完整结果，这些API容易出错，并且往往导致应用程序诉诸于未经充分测试的回退方案。

CSPRNG被认为已经足够好了，在大多数系统的大多数情况下可以提供实际上无限的随机数据。
唯一例外的情况是系统刚启动，尚未收集到足够的熵来初始化其CSPRNG。
在这些情况下，此API的设计理念是让实现(implementation)来应对这个问题，而不是把责任交给应用程序。

### 如果宿主平台上的随机性API较弱或者存在缺陷会发生什么情况？（What should happen on host platforms with weak or broken randomness APIs?）

处理这些情况是实现(implementations)的责任。
它们可以通过从其他来源收集的数据补充宿主平台API来实现这一点，
它们可以拒绝运行使用随机API(Random API)的程序，
或者在必要时，可以动态地使程序陷入以防止程序继续使用不良数据执行。

鼓励实现定期补种(reseeding)（如果宿主平台尚未这样做）。

### 是否应该有一个随机性资源，且API是否应该采取处理？（Should there be a randomness resource, and should the API take a handle?）

程序不需要知道使用的是*哪个*随机生成器，因为数据是随机且无法区分的(indistinguishable)。

使用随机API的WASI程序将会有特定的随机API导入，因为它们与用于通用型`stream`的导入不同。

### 是否应提供随机数据作为`stream`？（Should random data be provided as a `stream`?）

复用`stream`类型很有吸引力，但是希望能为此API的使用者提供实际随机数据，而不是可能被替换的任意流的内容，
因此统一此API与`stream`并没有用。

这也确保了使用随机API的程序可以通过其导入来识别，如前一个问题所述。

### 此API是否应该指定安全位？（Should the API specify a number of bits of security?）

最佳实践建议实现应至少提供196个安全性位。
然而，许多宿主平台的CSPRNG API目前并未记录其安全性位，
且不希望要求wasm引擎在已有CSPRNG的宿主平台上运行自己的CSPRNG，
因此目前API不指定具体的位数。

### 为什么insecure-random的返回值是固定大小？（Why is insecure-random a fixed-sized return value?）

这限制了通过它可获取的数据量。由于其不安全性，不建议将其作为`getrandom`的替代方案。

### 项目相关方利益 & 反馈（Stakeholder Interest & Feedback）

进入第3阶段之前的TODO。

Preview1具有随机函数，它们在工具链中广泛使用。

### 参考文献 & 致谢（References & acknowledgements）

非常感谢以下人员提供的宝贵反馈和建议：

- Zach Lym
- Luke Wagner
- Linux Weekly News' many articles about Linux random APIs including [this one].

[this one]: https://lwn.net/Articles/808575/
