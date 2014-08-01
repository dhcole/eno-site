---
published: true
layout: post
category: blog
title: "Writing TileMill&nbsp;Extensions&nbsp;in OSX"
permalink: /blog/writing-tilemill-plugins
class: tilemill-plugins
headline: images/blog/tilemill-plugins.png
---

<p>Writing a plugin for <a href="http://mapbox.com/tilemill">TileMill</a> is a great way to add features or additional functionality you’d like to see in the app. If you have a good understanding of how <a href="http://backbonejs.org">Backbone</a> works you’re well primed to stat authoring your own. For those not familiar with the plugins interface here’s a screenshot of what it looks like:</p>

<p><img src="http://cl.ly/LIVX/screenshot_2012-12-02Screen%20Shot%202012-12-02%20at%205.06.47%20PM.png" alt=""></p>

<p>The section under <em>available</em> is where you’ll find plugins in the wild (or otherwise not installed) and your’s too could be on that list for others to use but before I jump into building one let’s dive into TileMill.</p>

<div class="note">
<strong>Note:</strong> Plugins are not officially supported and may potentially break existing TileMill functionality. If you encounter bugs with TileMill, report them <a href="https://github.com/mapbox/tilemill/issues">to the issue tracker</a> but disable any custom plugins you may have and test again.
</div>

<h2 id="tilemillapp">TileMill.app</h2>

<p>To take a pen from <a href="http://mathiasbynens.be/notes/shell-script-mac-apps">Mathias Bynens’s post on creating Mac apps</a>: Mac applications have an .app extension and while it looks like a file it’s actually a package. You can view TileMIll’s application package by right-clicking Tilemill.app in finder it and choosing “Show Package Contents”.</p>

<p><img src="http://cl.ly/image/431x2V1m3m3g/screenshot_2012-11-05Screen%20Shot%202012-11-05%20at%208.02.40%20PM.png" alt="Show Package Contents osx"></p>

<p>The main application structure is in <code>Contents &gt; Resources</code> and it’s here you’ll find files that resemble <a href="https://github.com/mapbox/tilemill">TileMill’s master branch</a>. Have a look at how TileMill is structured. Like a traditional Backbone application the components that make up routing, data retrieval, or rendering are neatly decoupled and added as sing files in respective <em>controllers</em>, <em>models</em>, and <em>views</em> directories. Depending on what you want to do with your plugin, it’s these files you’ll likely want to scan through.</p>

<h2 id="setting-up-your-work-environment">Setting up your Work Environment</h2>

<ol>
  <li>
    <p>I recommend opening this <code>Resources</code> directory in your code editor as you begin writing your plugin as you can easily console.log anything while TileMill is running.</p>
  </li>
  <li>
    <p>You’ll also want to run the TileMill in Chrome over of its GUI to take advantage of DevTools for debugging. You can do this by navigating to <a href="http://localhost:20009">localhost:20009</a>.<img src="http://cl.ly/image/2l3L00080j1e/screenshot_2012-12-02Screen%20Shot%202012-12-02%20at%201.26.46%20PM.png" alt=""></p>
  </li>
  <li>
    <p>Lastly, open the users plugins directory by navigating to <code>~/.tilemill/node_modules</code> If you’ve downloaded any additional third party plugins they’ll show up there. Authoring a plugin should be made here before you publish to the world.</p>
  </li>
</ol>

<p><img src="http://cl.ly/image/2K0C1Y2x1w3x/screenshot_2012-11-06Screen%20Shot%202012-11-06%20at%209.58.29%20AM.png" alt=""></p>

<h2 id="the-elements-of-a-plugin">The elements of a plugin</h2>

<p>To get started, I recommend <a href="https://github.com/tristen/tilemill-tablesort">checking out the tilemill-tablesort plugin</a> I wrote for reference. It’s goal is to provide sorting functionality to the features data table.</p>

<h3 id="packagejson">package.json</h3>

<p>A plugin really only has one requirement: a <code>package.json</code> file. Mine looks like this:</p>

<div class="highlight"><pre><code class="language-js" data-lang="js"><span class="p">{</span>
    <span class="s2">"name"</span><span class="o">:</span> <span class="s2">"tilemill-tablesort"</span><span class="p">,</span>
    <span class="s2">"description"</span><span class="o">:</span> <span class="s2">"Sort columns on data tables"</span><span class="p">,</span>
    <span class="s2">"version"</span><span class="o">:</span> <span class="s2">"0.1.0"</span><span class="p">,</span>
    <span class="s2">"engines"</span><span class="o">:</span> <span class="p">{</span>
    <span class="s2">"tilemill"</span><span class="o">:</span> <span class="s2">"~0.10"</span>
    <span class="p">},</span>
    <span class="s2">"keywords"</span><span class="o">:</span> <span class="p">[</span><span class="s2">"tilemill"</span><span class="p">]</span>
<span class="p">}</span></code></pre></div>

<p>TileMill scans the npm registry to find packages that contain <strong>tilemill</strong> (available to it’s version) in its engines object and populates its listing on TileMill’s plugin page with the name and description. The version property has an important value as each time you re-publish your plugin with new changes TileMill will pull in this new version and prompt a user to upgrade.</p>

<h3 id="plugin-assets">Plugin assets</h3>

<p>Plugins with any UI feature we’ll likely contain <code>.css</code> or <code>.js</code> files. In a file I’ve named <code>servers/Route.bones</code> I reference a script that pushes tilemill-tablesort.css into TileMill’s assets/styles array and tablesort.min.js file into the assets/scripts array:</p>

<div class="highlight"><pre><code class="language-js" data-lang="js"><span class="kd">var</span> <span class="nx">assets</span> <span class="o">=</span> <span class="nx">servers</span><span class="p">[</span><span class="s1">'Route'</span><span class="p">].</span><span class="nx">prototype</span><span class="p">.</span><span class="nx">assets</span><span class="p">;</span>
<span class="nx">assets</span><span class="p">.</span><span class="nx">styles</span><span class="p">.</span><span class="nx">push</span><span class="p">(</span><span class="nx">require</span><span class="p">.</span><span class="nx">resolve</span><span class="p">(</span><span class="s1">'../assets/tilemill-tablesort.css'</span><span class="p">));</span>
<span class="nx">assets</span><span class="p">.</span><span class="nx">scripts</span><span class="p">.</span><span class="nx">push</span><span class="p">(</span><span class="nx">require</span><span class="p">.</span><span class="nx">resolve</span><span class="p">(</span><span class="s1">'../assets/tablesort.min.js'</span><span class="p">));</span></code></pre></div>

<h3 id="extending-existing-views">Extending existing views</h3>

<p>For plugins that provide additional functionality to TileMill’s native interface you’ll want to extend native code - not duplicate it. Have a look at how my <code>views/Datasource.js</code> file looks:</p>

<div class="highlight"><pre><code class="language-js" data-lang="js"><span class="nx">view</span> <span class="o">=</span> <span class="nx">views</span><span class="p">.</span><span class="nx">Datasource</span><span class="p">.</span><span class="nx">extend</span><span class="p">();</span>

<span class="nx">view</span><span class="p">.</span><span class="nx">prototype</span><span class="p">.</span><span class="nx">render</span> <span class="o">=</span> <span class="nx">_</span><span class="p">.</span><span class="nx">wrap</span><span class="p">(</span><span class="nx">view</span><span class="p">.</span><span class="nx">prototype</span><span class="p">.</span><span class="nx">render</span><span class="p">,</span> <span class="kd">function</span><span class="p">(</span><span class="nx">func</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">func</span><span class="p">.</span><span class="nx">call</span><span class="p">(</span><span class="k">this</span><span class="p">);</span>
    <span class="nx">$</span><span class="p">(</span><span class="s1">'table'</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">el</span><span class="p">).</span><span class="nx">attr</span><span class="p">(</span><span class="s1">'id'</span><span class="p">,</span> <span class="s1">'features'</span><span class="p">);</span>
    <span class="nx">$</span><span class="p">(</span><span class="s1">'tr.min, tr.max'</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">el</span><span class="p">).</span><span class="nx">addClass</span><span class="p">(</span><span class="s1">'no-sort'</span><span class="p">);</span>

    <span class="k">this</span><span class="p">.</span><span class="nx">sort</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Tablesort</span><span class="p">(</span><span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="s1">'features'</span><span class="p">));</span>
<span class="p">});</span>

<span class="nx">view</span><span class="p">.</span><span class="nx">prototype</span><span class="p">.</span><span class="nx">showAll</span> <span class="o">=</span> <span class="nx">_</span><span class="p">.</span><span class="nx">wrap</span><span class="p">(</span><span class="nx">view</span><span class="p">.</span><span class="nx">prototype</span><span class="p">.</span><span class="nx">showAll</span><span class="p">,</span> <span class="kd">function</span><span class="p">(</span><span class="nx">func</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">func</span><span class="p">.</span><span class="nx">call</span><span class="p">(</span><span class="k">this</span><span class="p">);</span>
    <span class="k">this</span><span class="p">.</span><span class="nx">sort</span><span class="p">.</span><span class="nx">refresh</span><span class="p">();</span>
    <span class="k">return</span> <span class="kc">false</span><span class="p">;</span>
<span class="p">});</span></code></pre></div>

<p>Backbone provides an <a href="http://backbonejs.org/#View-extend">extend</a> helper method that allows you to bring over the same methods from an existing constructor - In this case, TileMill’s already defined view.Datasource. In my view I’m adding new functionality to two existing methods: <em>render</em> and <em>showAll</em>. To ensure the functionality of those methods are carried over into my plugin I use <a href="http://underscorejs.org/#wrap">Underscore’s wrap</a> to add my own code to this existing function after its been run. Pass the function in a callback be sure to call it within this new function:</p>

<div class="highlight"><pre><code class="language-js" data-lang="js"><span class="nx">func</span><span class="p">.</span><span class="nx">call</span><span class="p">(</span><span class="k">this</span><span class="p">);</span></code></pre></div>

<p>The rest of the code is now real authoring and specific to table sorting.</p>

<h2 id="publishing">Publishing</h2>

<p>To make your plugin available for others to use you’ll need to publish it. Assuming you have <a href="npmjs.org">npm</a> installed, (<em>hint:</em> this comes bundled with <a href="http://nodejs.org">node.js</a>) create an npm user account if you haven’t done so already by typing <code>npm adduser</code> in terminal and following the prompts. The last step is to execute this command in your plugins root directory:</p>

<div class="highlight"><pre><code class="language-bash" data-lang="bash">npm publ</code></pre></div>

<p>And you’re all set. Feeling uninspired? There are lots of <a href="https://github.com/mapbox/tilemill/issues?labels=plugins&amp;page=1&amp;state=open">good feature requests that come up</a>.</p>
