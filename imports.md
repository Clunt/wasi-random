<h1><a name="imports">World imports</a></h1>
<ul>
<li>Imports:
<ul>
<li>interface <a href="#wasi_random_random_0_2_0"><code>wasi:random/random@0.2.0</code></a></li>
<li>interface <a href="#wasi_random_insecure_0_2_0"><code>wasi:random/insecure@0.2.0</code></a></li>
<li>interface <a href="#wasi_random_insecure_seed_0_2_0"><code>wasi:random/insecure-seed@0.2.0</code></a></li>
</ul>
</li>
</ul>
<h2><a name="wasi_random_random_0_2_0"></a>Import interface wasi:random/random@0.2.0</h2>
<p>WASI Random是随机数据API。</p>
<p>它旨在至少在Unix系列平台和Windows之间具有可移植性。</p>
<hr />
<h3>Functions</h3>
<h4><a name="get_random_bytes"></a><code>get-random-bytes: func</code></h4>
<p>返回长度为<code>len</code>的密码安全随机(cryptographically-secure random)或伪随机(pseudo-random)字节。</p>
<p>此函数必须生成至少与足够种子的密码安全伪随机数生成器（CSPRNG）一样的密码安全且快速的数据。
从调用程序的视角看，在任何情况下它都不能阻塞，包括第一次请求和请求字节数时。
返回的数据必须始终是不可预测的。</p>
<p>这个函数必须始终返回最新数据。
确定性环境必须省略此函数，而不是使用确定性数据实现它。</p>
<h5>Params</h5>
<ul>
<li><a name="get_random_bytes.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="get_random_bytes.0"></a> list&lt;<code>u8</code>&gt;</li>
</ul>
<h4><a name="get_random_u64"></a><code>get-random-u64: func</code></h4>
<p>返回密码安全随机或伪随机的<code>u64</code>值。</p>
<p>此函数返回与<a href="#get_random_bytes"><code>get-random-bytes</code></a>相同类型的数据，表示为<code>u64</code>。</p>
<h5>Return values</h5>
<ul>
<li><a name="get_random_u64.0"></a> <code>u64</code></li>
</ul>
<h2><a name="wasi_random_insecure_0_2_0"></a>Import interface wasi:random/insecure@0.2.0</h2>
<p>insecure接口用于不安全的伪随机数。</p>
<p>它旨在至少在Unix系列平台和Windows之间具有可移植性。</p>
<hr />
<h3>Functions</h3>
<h4><a name="get_insecure_random_bytes"></a><code>get-insecure-random-bytes: func</code></h4>
<p>返回长度为<code>len</code>的不安全伪随机字节。</p>
<p>此函数不具备密码安全性。
请勿将其用于任何安全相关事项。</p>
<p>返回的字节的值没有具体的要求，
但是建议实现应返回均匀分布且具有长周期的值。</p>
<h5>Params</h5>
<ul>
<li><a name="get_insecure_random_bytes.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="get_insecure_random_bytes.0"></a> list&lt;<code>u8</code>&gt;</li>
</ul>
<h4><a name="get_insecure_random_u64"></a><code>get-insecure-random-u64: func</code></h4>
<p>返回不安全伪随机的<code>u64</code>值。Return an insecure pseudo-random <code>u64</code> value.</p>
<p>此函数返回与<a href="#get_insecure_random_bytes"><code>get-insecure-random-bytes</code></a>相同类型的数据，表示为<code>u64</code>。</p>
<h5>Return values</h5>
<ul>
<li><a name="get_insecure_random_u64.0"></a> <code>u64</code></li>
</ul>
<h2><a name="wasi_random_insecure_seed_0_2_0"></a>Import interface wasi:random/insecure-seed@0.2.0</h2>
<p>insecure-seed接口用于播种(seeding)哈希映射(hash-map)DoS抵御(resistance)。</p>
<p>它旨在至少在Unix系列平台和Windows之间具有可移植性。</p>
<hr />
<h3>Functions</h3>
<h4><a name="insecure_seed"></a><code>insecure-seed: func</code></h4>
<p>返回可能包含伪随机值的128位值。</p>
<p>返回的值不要求由CSPRNG计算，并且甚至可以是完全确定性的。
鼓励宿主实现为任何暴露给攻击者已控内容(attacker-controlled content)的程序提供伪随机值，
以启用许多语言哈希映射实现中的DoS防护。</p>
<p>这个函数预期只被调用一次，由源语言来初始化其哈希映射实现中的服务拒绝(Denial Of Service，DoS)保护。</p>
<h1>预计的未来演变</h1>
<p>这可能会被改变为一个值导入，以防止它被多次调用，并且可能被用于除了DoS保护之外的目的。</p>
<h5>Return values</h5>
<ul>
<li><a name="insecure_seed.0"></a> (<code>u64</code>, <code>u64</code>)</li>
</ul>
