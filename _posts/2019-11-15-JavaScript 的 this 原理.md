---
layout: post
title:  JavaScript 的 this 原理
date:   2019-11-15 1:05:00
catalog:  true
tags:
    - JavaScript 
    - this
         
       

---

<div class="asset-content entry-content" id="main-content">

                                    <!-- div class="asset-body" -->
                                        <h2>一、问题的由来</h2>

<p>学懂 JavaScript 语言，一个标志就是理解下面两种写法，可能有不一样的结果。</p> 

                                    <!-- /div -->


                                    <!-- div id="more" class="asset-more" -->
                                        <blockquote><pre class=" language-javascript"><code class=" language-javascript">
<span class="token keyword">var</span> obj <span class="token operator">=</span> <span class="token punctuation">{</span>
  foo<span class="token punctuation">:</span> <span class="token keyword">function</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>

<span class="token keyword">var</span> foo <span class="token operator">=</span> obj<span class="token punctuation">.</span>foo<span class="token punctuation">;</span>
<span class="token comment" spellcheck="true">
// 写法一
</span>obj<span class="token punctuation">.</span><span class="token function">foo<span class="token punctuation">(</span></span><span class="token punctuation">)</span>
<span class="token comment" spellcheck="true">
// 写法二
</span><span class="token function">foo<span class="token punctuation">(</span></span><span class="token punctuation">)</span>
</code></pre></blockquote>

<p>上面代码中，虽然<code>obj.foo</code>和<code>foo</code>指向同一个函数，但是执行结果可能不一样。请看下面的例子。</p>

<blockquote><pre class=" language-javascript"><code class=" language-javascript">
<span class="token keyword">var</span> obj <span class="token operator">=</span> <span class="token punctuation">{</span>
  foo<span class="token punctuation">:</span> <span class="token keyword">function</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span> console<span class="token punctuation">.</span><span class="token function">log<span class="token punctuation">(</span></span>this<span class="token punctuation">.</span>bar<span class="token punctuation">)</span> <span class="token punctuation">}</span><span class="token punctuation">,</span>
  bar<span class="token punctuation">:</span> <span class="token number">1</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>

<span class="token keyword">var</span> foo <span class="token operator">=</span> obj<span class="token punctuation">.</span>foo<span class="token punctuation">;</span>
<span class="token keyword">var</span> bar <span class="token operator">=</span> <span class="token number">2</span><span class="token punctuation">;</span>

obj<span class="token punctuation">.</span><span class="token function">foo<span class="token punctuation">(</span></span><span class="token punctuation">)</span><span class="token comment" spellcheck="true"> // 1
</span><span class="token function">foo<span class="token punctuation">(</span></span><span class="token punctuation">)</span><span class="token comment" spellcheck="true"> // 2
</span></code></pre></blockquote>

<p>这种差异的原因，就在于函数体内部使用了<code>this</code>关键字。很多教科书会告诉你，<code>this</code>指的是函数运行时所在的环境。对于<code>obj.foo()</code>来说，<code>foo</code>运行在<code>obj</code>环境，所以<code>this</code>指向<code>obj</code>；对于<code>foo()</code>来说，<code>foo</code>运行在全局环境，所以<code>this</code>指向全局环境。所以，两者的运行结果不一样。</p>

<p>这种解释没错，但是教科书往往不告诉你，为什么会这样？也就是说，函数的运行环境到底是怎么决定的？举例来说，为什么<code>obj.foo()</code>就是在<code>obj</code>环境执行，而一旦<code>var foo = obj.foo</code>，<code>foo()</code>就变成在全局环境执行？</p>

<p>本文就来解释 JavaScript 这样处理的原理。理解了这一点，你就会彻底理解<code>this</code>的作用。</p>

<h2>二、内存的数据结构</h2>

<p>JavaScript 语言之所以有<code>this</code>的设计，跟内存里面的数据结构有关系。</p>

<blockquote><pre class=" language-javascript"><code class=" language-javascript">
<span class="token keyword">var</span> obj <span class="token operator">=</span> <span class="token punctuation">{</span> foo<span class="token punctuation">:</span>  <span class="token number">5</span> <span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre></blockquote>

<p>上面的代码将一个对象赋值给变量<code>obj</code>。JavaScript 引擎会先在内存里面，生成一个对象<code>{ foo: 5 }</code>，然后把这个对象的内存地址赋值给变量<code>obj</code>。</p>

<p><img src="https://www.wangbase.com/blogimg/asset/201806/bg2018061801.png" alt="" title=""></p>

<p>也就是说，变量<code>obj</code>是一个地址（reference）。后面如果要读取<code>obj.foo</code>，引擎先从<code>obj</code>拿到内存地址，然后再从该地址读出原始的对象，返回它的<code>foo</code>属性。</p>

<p>原始的对象以字典结构保存，每一个属性名都对应一个属性描述对象。举例来说，上面例子的<code>foo</code>属性，实际上是以下面的形式保存的。</p>

<p><img src="https://www.wangbase.com/blogimg/asset/201806/bg2018061802.png" alt="" title=""></p>

<blockquote><pre class=" language-javascript"><code class=" language-javascript">
<span class="token punctuation">{</span>
  foo<span class="token punctuation">:</span> <span class="token punctuation">{</span>
    <span class="token punctuation">[</span><span class="token punctuation">[</span>value<span class="token punctuation">]</span><span class="token punctuation">]</span><span class="token punctuation">:</span> <span class="token number">5</span>
    <span class="token punctuation">[</span><span class="token punctuation">[</span>writable<span class="token punctuation">]</span><span class="token punctuation">]</span><span class="token punctuation">:</span> <span class="token boolean">true</span>
    <span class="token punctuation">[</span><span class="token punctuation">[</span>enumerable<span class="token punctuation">]</span><span class="token punctuation">]</span><span class="token punctuation">:</span> <span class="token boolean">true</span>
    <span class="token punctuation">[</span><span class="token punctuation">[</span>configurable<span class="token punctuation">]</span><span class="token punctuation">]</span><span class="token punctuation">:</span> <span class="token boolean">true</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre></blockquote>

<p>注意，<code>foo</code>属性的值保存在属性描述对象的<code>value</code>属性里面。</p>

<h2>三、函数</h2>

<p>这样的结构是很清晰的，问题在于属性的值可能是一个函数。</p>

<blockquote><pre class=" language-javascript"><code class=" language-javascript">
<span class="token keyword">var</span> obj <span class="token operator">=</span> <span class="token punctuation">{</span> foo<span class="token punctuation">:</span> <span class="token keyword">function</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span> <span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre></blockquote>

<p>这时，引擎会将函数单独保存在内存中，然后再将函数的地址赋值给<code>foo</code>属性的<code>value</code>属性。</p>

<p><img src="https://www.wangbase.com/blogimg/asset/201806/bg2018061803.png" alt="" title=""></p>

<blockquote><pre class=" language-javascript"><code class=" language-javascript">
<span class="token punctuation">{</span>
  foo<span class="token punctuation">:</span> <span class="token punctuation">{</span>
    <span class="token punctuation">[</span><span class="token punctuation">[</span>value<span class="token punctuation">]</span><span class="token punctuation">]</span><span class="token punctuation">:</span> 函数的地址
    <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre></blockquote>

<p>由于函数是一个单独的值，所以它可以在不同的环境（上下文）执行。</p>

<blockquote><pre class=" language-javascript"><code class=" language-javascript">
<span class="token keyword">var</span> f <span class="token operator">=</span> <span class="token keyword">function</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">}</span><span class="token punctuation">;</span>
<span class="token keyword">var</span> obj <span class="token operator">=</span> <span class="token punctuation">{</span> f<span class="token punctuation">:</span> f <span class="token punctuation">}</span><span class="token punctuation">;</span>
<span class="token comment" spellcheck="true">
// 单独执行
</span><span class="token function">f<span class="token punctuation">(</span></span><span class="token punctuation">)</span>
<span class="token comment" spellcheck="true">
// obj 环境执行
</span>obj<span class="token punctuation">.</span><span class="token function">f<span class="token punctuation">(</span></span><span class="token punctuation">)</span>
</code></pre></blockquote>

<h2>四、环境变量</h2>

<p>JavaScript 允许在函数体内部，引用当前环境的其他变量。</p>

<blockquote><pre class=" language-javascript"><code class=" language-javascript">
<span class="token keyword">var</span> f <span class="token operator">=</span> <span class="token keyword">function</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  console<span class="token punctuation">.</span><span class="token function">log<span class="token punctuation">(</span></span>x<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre></blockquote>

<p>上面代码中，函数体里面使用了变量<code>x</code>。该变量由运行环境提供。</p>

<p>现在问题就来了，由于函数可以在不同的运行环境执行，所以需要有一种机制，能够在函数体内部获得当前的运行环境（context）。所以，<code>this</code>就出现了，它的设计目的就是在函数体内部，指代函数当前的运行环境。</p>

<blockquote><pre class=" language-javascript"><code class=" language-javascript">
<span class="token keyword">var</span> f <span class="token operator">=</span> <span class="token keyword">function</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  console<span class="token punctuation">.</span><span class="token function">log<span class="token punctuation">(</span></span>this<span class="token punctuation">.</span>x<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre></blockquote>

<p>上面代码中，函数体里面的<code>this.x</code>就是指当前运行环境的<code>x</code>。</p>

<blockquote><pre class=" language-javascript"><code class=" language-javascript">
<span class="token keyword">var</span> f <span class="token operator">=</span> <span class="token keyword">function</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  console<span class="token punctuation">.</span><span class="token function">log<span class="token punctuation">(</span></span>this<span class="token punctuation">.</span>x<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>

<span class="token keyword">var</span> x <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">;</span>
<span class="token keyword">var</span> obj <span class="token operator">=</span> <span class="token punctuation">{</span>
  f<span class="token punctuation">:</span> f<span class="token punctuation">,</span>
  x<span class="token punctuation">:</span> <span class="token number">2</span><span class="token punctuation">,</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
<span class="token comment" spellcheck="true">
// 单独执行
</span><span class="token function">f<span class="token punctuation">(</span></span><span class="token punctuation">)</span><span class="token comment" spellcheck="true"> // 1
</span><span class="token comment" spellcheck="true">
// obj 环境执行
</span>obj<span class="token punctuation">.</span><span class="token function">f<span class="token punctuation">(</span></span><span class="token punctuation">)</span><span class="token comment" spellcheck="true"> // 2
</span></code></pre></blockquote>

<p>上面代码中，函数<code>f</code>在全局环境执行，<code>this.x</code>指向全局环境的<code>x</code>。</p>

<p><img src="https://www.wangbase.com/blogimg/asset/201806/bg2018061804.png" alt="" title=""></p>

<p>在<code>obj</code>环境执行，<code>this.x</code>指向<code>obj.x</code>。</p>

<p><img src="https://www.wangbase.com/blogimg/asset/201806/bg2018061805.png" alt="" title=""></p>

<p>回到本文开头提出的问题，<code>obj.foo()</code>是通过<code>obj</code>找到<code>foo</code>，所以就是在<code>obj</code>环境执行。一旦<code>var foo = obj.foo</code>，变量<code>foo</code>就直接指向函数本身，所以<code>foo()</code>就变成在全局环境执行。</p>

<p>（完）</p>

                                    <!-- /div -->

                                </div>

