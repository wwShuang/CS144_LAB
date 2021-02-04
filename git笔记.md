---


---

<blockquote>
<p><a href="https://www.youtube.com/watch?v=nQDFBd5NFA8">github 创建ssh key</a><br>
<a href="https://levelup.gitconnected.com/how-to-add-ssh-to-github-account-on-linux-1c05c71dacc9">额外参考</a></p>
</blockquote>
<blockquote>
<p><a href="https://git-scm.com/doc">git 官方手册</a><br>
<a href="https://blog.csdn.net/Lucky_LXG/article/details/77849212">git使用</a><br>
<a href="https://blog.csdn.net/top_code/article/details/51931916">branch</a></p>
</blockquote>
<p>总结：<br>
我通常push步骤，添加完文件，实验结束后</p>
<ol>
<li>查看本地分支 <code>git branch</code>， * 是当前的分支</li>
<li>是否创建新分支, 需要则<code>git branch [branch name]</code></li>
<li>查看当前分支状态 <code>git status</code></li>
<li>添加 所有新文件 <code>git add .</code></li>
<li><code>git commit -m [branch name]</code></li>
<li>push 到远程<code>git push origin [branch name]</code>（远程没有当前新branch 会自动创建）</li>
</ol>
<p>不知道符不符合常规操作…</p>

