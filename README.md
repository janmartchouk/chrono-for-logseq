<p align='center'><img src='src/style/logo.svg' width=250px/></p>

<h2>A pretty pretty timeline based on your Logseq notes</h2>
<img src="https://github.com/janmartchouk/chrono-for-logseq/assets/19735475/4cea77eb-ef03-485c-8103-1d35551f51f3" />
<h1>
<a target='_blank' href="https://martcho.uk/app/chrono/?init=true&disabled_categories=%7B%22people%22%3Afalse%2C%22events%22%3Afalse%2C%22works%22%3Atrue%7D&entries=%5B%5B%22Lukrez%22%2C-99%2C-53%2C%22people%22%5D%2C%5B%22Alexander+der+gro%C3%9Fe%22%2C-356%2C-323%2C%22people%22%5D%2C%5B%22Epikur%22%2C-341%2C-271%2C%22people%22%5D%2C%5B%22Pyrrhon+von+Elis%22%2C-362%2C-275%2C%22people%22%5D%2C%5B%22Platon%22%2C-428%2C-347%2C%22people%22%5D%2C%5B%22Cicero%22%2C-106%2C-43%2C%22people%22%5D%2C%5B%22Panaitios+von+Rhodos%22%2C-180%2C-110%2C%22people%22%5D%2C%5B%22Der+Staat%22%2C-375%2C-275%2C%22works%22%5D%2C%5B%22Hellenismus%22%2C-323%2C-30%2C%22events%22%5D%2C%5B%22R%C3%B6mische+Republik%22%2C-509%2C-31%2C%22events%22%5D%2C%5B%22Aristoteles%22%2C-384%2C-322%2C%22people%22%5D%2C%5B%22Zenon+von+Kition%22%2C-333%2C-262%2C%22people%22%5D%2C%5B%22K%C3%B6nigszeit+%28Rom%29%22%2C-753%2C-509%2C%22events%22%5D%2C%5B%22Sokrates%22%2C-469%2C-399%2C%22people%22%5D%2C%5B%22Pelopponesischer+Krieg%22%2C-431%2C-404%2C%22events%22%5D%2C%5B%22Pythagoras%22%2C-570%2C-510%2C%22people%22%5D%2C%5B%22De+rerum+Natura%22%2C-1%2C99%2C%22works%22%5D%2C%5B%22Mark+Aurel%22%2C121%2C180%2C%22people%22%5D%2C%5B%22Krates+von+Theben%22%2C-365%2C-285%2C%22people%22%5D%2C%5B%22R%C3%B6misches+Reich%22%2C-753%2C476%2C%22events%22%5D%2C%5B%22Attischer+Seebund%22%2C-478%2C-404%2C%22events%22%5D%2C%5B%22Thales%22%2C-624%2C-548%2C%22people%22%5D%2C%5B%22Kaiserzeit+in+Rom%22%2C-31%2C476%2C%22events%22%5D%2C%5B%22Seneca%22%2C1%2C65%2C%22people%22%5D%2C%5B%22Antisthenes%22%2C-445%2C-365%2C%22people%22%5D%2C%5B%22Nero%22%2C37%2C68%2C%22people%22%5D%2C%5B%22Diogenes+von+Sinope%22%2C-413%2C-323%2C%22people%22%5D%5D">Demo</a> 
&nbsp;&nbsp;&nbsp;
<a target='_blank' href="https://martcho.uk/app/chrono/">Public Instance</a>
</h1>
<br>
<h2>What is this?</h2>
<p>
  <a target='_blank' href="https://logseq.com/">Logseq</a> is a free and open-source note-taking app that, conveniently, stores notes in markdown format. This app, Chrono, takes your Logseq notes and searches for pages about people (which include a 'birth' and maybe a 'death' <a target='_blank' href="https://discuss.logseq.com/t/lesson-5-how-to-power-your-workflows-using-properties-and-dynamic-variables/10173">property</a>), events (which include a 'start' and maybe an 'end' property") and works (such as books, which include a 'date' property). How exactly your properties are named is configurable in Chrono. Then, Chrono displays these things on a neat timeline, which (I have found) helps a lot with visualizing temporal relations between things you have noted (for example, that Epikur lived at the same time as Zenon, meaning Stoicism and Epikureism developed at the same time. neat.) 
</p>
<p float='left'>
<img src="https://github.com/janmartchouk/chrono-for-logseq/assets/19735475/084cac04-093f-467e-b3bc-e414e522b8c6" width=49% style='display:inline' />
<img src="https://github.com/janmartchouk/chrono-for-logseq/assets/19735475/386de892-717c-486c-abb2-3eef603af370" width=49% style='display:inline'/>
</p>
<h2>Self-hosting</h2>
<p>If you don't trust my public instance, you can self-host this with <a target='_blank' href='https://yarnpkg.com/'>yarn</a>. Simply:
<ol>
  <li>check out the repo (main branch)</li> 
  <li>install dependencies using <code>yarn install</code></li>
  <li>and run a live version using <code>yarn run dev</code></li>
</ol>
</p>
<p>
  This project was built using <a target='_blank' href='https://svelte.dev/'>Svelte</a> by <a target='_blank' href='https://martcho.uk/'>me</a>. Thanks Svelte!
</p>
