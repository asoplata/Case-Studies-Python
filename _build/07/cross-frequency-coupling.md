---
interact_link: content/07/cross-frequency-coupling.ipynb
kernel_name: python3
kernel_path: content/07
has_widgets: false
title: |-
  Cross Frequency Coupling
pagenum: 10
prev_page:
  url: /06/filtering-scalp-eeg.html
next_page:
  url: /08/basic-visualizations-and-descriptive-statistics-of-spike-train-data.html
suffix: .ipynb
search: frequency phase amplitude x signal data transform t hilbert cfc omega e h high low f div lfp pi analysis step id coupling between band lets analytic activity domain mbox inspection compute shift class question rhythms visual consider filter bands plot frequencies value cross different extract filtered signals begin y vector s hz python variable function our q example envelope spectrum original z complex array surrogate spectral does computing here assess interest negative real positive resampling appendix not observed rhythm note case abbr results its end tau instantaneous into determine related code computed www ncbi nlm nih gov techniques interval

comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---

    <main class="jupyter-page">
    <div id="page-info"><div id="page-title">Cross Frequency Coupling</div>
</div>
    <div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h1 id="Cross-frequency-coupling-for-the-practicing-neuroscientist">Cross-frequency coupling <em>for the practicing neuroscientist</em><a class="anchor-link" href="#Cross-frequency-coupling-for-the-practicing-neuroscientist"> </a></h1>
</div>
</div>
</div>
</div>

<div class="jb_cell tag_Question">

<div class="cell border-box-sizing text_cell rendered tag_Question"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<div class="question">

_**Synopsis**_ 

**Data:** 100 s of local field potential data sampled at 1000 Hz.

**Goal:** Characterize the coupling between rhythms of different frequency.

**Tools:** Hilbert transform, analytic signal, instantaneous phase, cross-frequency coupling.

</div>
</div>
</div>
</div>
</div>

<div class="jb_cell tag_Hard">

<div class="cell border-box-sizing text_cell rendered tag_Hard"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<ul>
<li><a href="#introduction">Introduction</a></li>
<li><a href="#data-analysis">Data Analysis</a><ul>
<li><a href="#visual-inspection">Visual inspection</a></li>
<li><a href="#spectral-analysis">Spectral analysis</a></li>
<li><a href="#cfc">Cross frequency coupling</a><ol>
<li><a href="#step1">Filter the data into high- and low-frequency bands.</a></li>
<li><a href="#step2">Extract the amplitude and phase from the filtered signals.</a><ul>
<li><a href="#hilbert">What does the Hilbert transform do?</a></li>
</ul>
</li>
<li><a href="#step3">Determine if the phase and amplitude are related</a><ul>
<li><a href="#m1">The phase-amplitude plot</a> </li>
</ul>
</li>
</ol>
</li>
</ul>
</li>
<li><a href="#summary">Summary</a></li>
<li><a href="#appendix">Appendix</a></li>
</ul>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="On-ramp:-computing-cross-frequency-coupling-in-Python">On-ramp: computing cross-frequency coupling in Python<a class="anchor-link" href="#On-ramp:-computing-cross-frequency-coupling-in-Python"> </a></h2><p>We begin this module with an "<em>on-ramp</em>" to analysis. The purpose of this on-ramp is to introduce you immediately to a core concept in this module: how to compute cross-frequency coupling (CFC) in Python. You may not understand all aspects of the program here, but that's not the point. Instead, the purpose of this on-ramp is to  illustrate what <em>can</em> be done. Our advice is to simply run the code below and see what happens ...</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="kn">from</span> <span class="nn">scipy.io</span> <span class="kn">import</span> <span class="n">loadmat</span>
<span class="kn">from</span> <span class="nn">scipy</span> <span class="kn">import</span> <span class="n">signal</span>
<span class="kn">import</span> <span class="nn">numpy</span> <span class="k">as</span> <span class="nn">np</span>
<span class="kn">import</span> <span class="nn">matplotlib.pyplot</span> <span class="k">as</span> <span class="nn">plt</span>

<span class="n">data</span> <span class="o">=</span> <span class="n">loadmat</span><span class="p">(</span><span class="s1">&#39;LFP-1.mat&#39;</span><span class="p">)</span>          <span class="c1"># Load the LFP data, </span>
<span class="n">t</span> <span class="o">=</span> <span class="n">data</span><span class="p">[</span><span class="s1">&#39;t&#39;</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span>                     <span class="c1"># ... extract t, the time variable,</span>
<span class="n">LFP</span> <span class="o">=</span> <span class="n">data</span><span class="p">[</span><span class="s1">&#39;LFP&#39;</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span>                 <span class="c1"># ... and LFP, the voltage variable,</span>
<span class="n">dt</span> <span class="o">=</span> <span class="n">t</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span> <span class="o">-</span> <span class="n">t</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span>                     <span class="c1"># Define the sampling interval,</span>
<span class="n">fNQ</span> <span class="o">=</span> <span class="mi">1</span> <span class="o">/</span> <span class="n">dt</span> <span class="o">/</span> <span class="mi">2</span>                     <span class="c1"># ... and Nyquist frequency. </span>

<span class="n">Wn</span> <span class="o">=</span> <span class="p">[</span><span class="mi">5</span><span class="p">,</span><span class="mi">7</span><span class="p">];</span>                          <span class="c1"># Set the passband [5-7] Hz,</span>
<span class="n">n</span> <span class="o">=</span> <span class="mi">100</span><span class="p">;</span>                             <span class="c1"># ... and filter order,</span>
<span class="n">b</span> <span class="o">=</span> <span class="n">signal</span><span class="o">.</span><span class="n">firwin</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">Wn</span><span class="p">,</span> <span class="n">nyq</span><span class="o">=</span><span class="n">fNQ</span><span class="p">,</span> <span class="n">pass_zero</span><span class="o">=</span><span class="kc">False</span><span class="p">,</span> <span class="n">window</span><span class="o">=</span><span class="s1">&#39;hamming&#39;</span><span class="p">);</span>
<span class="n">Vlo</span> <span class="o">=</span> <span class="n">signal</span><span class="o">.</span><span class="n">filtfilt</span><span class="p">(</span><span class="n">b</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="n">LFP</span><span class="p">);</span>    <span class="c1"># ... and apply it to the data.</span>

<span class="n">Wn</span> <span class="o">=</span> <span class="p">[</span><span class="mi">80</span><span class="p">,</span> <span class="mi">120</span><span class="p">];</span>                      <span class="c1"># Set the passband [80-120] Hz,</span>
<span class="n">n</span> <span class="o">=</span> <span class="mi">100</span><span class="p">;</span>                             <span class="c1"># ... and filter order,</span>
<span class="n">b</span> <span class="o">=</span> <span class="n">signal</span><span class="o">.</span><span class="n">firwin</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">Wn</span><span class="p">,</span> <span class="n">nyq</span><span class="o">=</span><span class="n">fNQ</span><span class="p">,</span> <span class="n">pass_zero</span><span class="o">=</span><span class="kc">False</span><span class="p">,</span> <span class="n">window</span><span class="o">=</span><span class="s1">&#39;hamming&#39;</span><span class="p">);</span>
<span class="n">Vhi</span> <span class="o">=</span> <span class="n">signal</span><span class="o">.</span><span class="n">filtfilt</span><span class="p">(</span><span class="n">b</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="n">LFP</span><span class="p">);</span>    <span class="c1"># ... and apply it to the data.</span>

<span class="n">phi</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">angle</span><span class="p">(</span><span class="n">signal</span><span class="o">.</span><span class="n">hilbert</span><span class="p">(</span><span class="n">Vlo</span><span class="p">))</span>  <span class="c1"># Compute phase of low-freq signal</span>
<span class="n">amp</span> <span class="o">=</span> <span class="nb">abs</span><span class="p">(</span><span class="n">signal</span><span class="o">.</span><span class="n">hilbert</span><span class="p">(</span><span class="n">Vhi</span><span class="p">))</span>       <span class="c1"># Compute amplitude of high-freq signal</span>

<span class="n">p_bins</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">arange</span><span class="p">(</span><span class="o">-</span><span class="n">np</span><span class="o">.</span><span class="n">pi</span><span class="p">,</span><span class="n">np</span><span class="o">.</span><span class="n">pi</span><span class="p">,</span><span class="mf">0.1</span><span class="p">)</span> <span class="c1"># To compute CFC, define phase bins,</span>
<span class="n">a_mean</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">zeros</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">size</span><span class="p">(</span><span class="n">p_bins</span><span class="p">)</span><span class="o">-</span><span class="mi">1</span><span class="p">)</span> <span class="c1"># ... variable to hold the amplitude,</span>
<span class="n">p_mean</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">zeros</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">size</span><span class="p">(</span><span class="n">p_bins</span><span class="p">)</span><span class="o">-</span><span class="mi">1</span><span class="p">)</span> <span class="c1"># ... and variable to hold the phase.</span>
<span class="k">for</span> <span class="n">k</span> <span class="ow">in</span> <span class="nb">range</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">size</span><span class="p">(</span><span class="n">p_bins</span><span class="p">)</span><span class="o">-</span><span class="mi">1</span><span class="p">):</span>      <span class="c1"># For each phase bin,</span>
    <span class="n">pL</span> <span class="o">=</span> <span class="n">p_bins</span><span class="p">[</span><span class="n">k</span><span class="p">]</span>					    <span class="c1">#... get lower phase limit,</span>
    <span class="n">pR</span> <span class="o">=</span> <span class="n">p_bins</span><span class="p">[</span><span class="n">k</span><span class="o">+</span><span class="mi">1</span><span class="p">]</span>				    <span class="c1">#... get upper phase limit.</span>
    <span class="n">indices</span><span class="o">=</span><span class="p">(</span><span class="n">phi</span><span class="o">&gt;=</span><span class="n">pL</span><span class="p">)</span> <span class="o">&amp;</span> <span class="p">(</span><span class="n">phi</span><span class="o">&lt;</span><span class="n">pR</span><span class="p">)</span>	    <span class="c1">#Find phases falling in this bin,</span>
    <span class="n">a_mean</span><span class="p">[</span><span class="n">k</span><span class="p">]</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">mean</span><span class="p">(</span><span class="n">amp</span><span class="p">[</span><span class="n">indices</span><span class="p">])</span>	<span class="c1">#... compute mean amplitude,</span>
    <span class="n">p_mean</span><span class="p">[</span><span class="n">k</span><span class="p">]</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">mean</span><span class="p">([</span><span class="n">pL</span><span class="p">,</span> <span class="n">pR</span><span class="p">])</span>		<span class="c1">#... save center phase.</span>
<span class="n">plt</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">p_mean</span><span class="p">,</span> <span class="n">a_mean</span><span class="p">)</span>                <span class="c1">#Plot the phase versus amplitude,</span>
<span class="n">plt</span><span class="o">.</span><span class="n">ylabel</span><span class="p">(</span><span class="s1">&#39;High-frequency amplitude&#39;</span><span class="p">)</span>  <span class="c1">#... with axes labeled.</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlabel</span><span class="p">(</span><span class="s1">&#39;Low-frequency phase&#39;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">title</span><span class="p">(</span><span class="s1">&#39;CFC&#39;</span><span class="p">);</span>
</pre></div>

    </div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<div class="question">

**Q:** Try to read the code above. Can you see how it loads data, computes the phase and amplitude of the signals, and assess the cross-frequency coupling?

**A:** If you've never computed cross-frequency coupling before, that's an especially difficult question. Please continue on to learn this **and more**!

</div>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="Introduction">Introduction<a id="introduction" /><a class="anchor-link" href="#Introduction"> </a></h2>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="kn">from</span> <span class="nn">IPython.lib.display</span> <span class="kn">import</span> <span class="n">YouTubeVideo</span>
<span class="n">YouTubeVideo</span><span class="p">(</span><span class="s1">&#39;Q-VQaY6iDMM&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">


<div class="output_html rendered_html output_subarea output_execute_result">

<iframe
    width="400"
    height="300"
    src="https://www.youtube.com/embed/Q-VQaY6iDMM"
    frameborder="0"
    allowfullscreen
></iframe>

</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="Background">Background<a class="anchor-link" href="#Background"> </a></h3><p>In this module, we focus on local field potential (LFP) recordings. The LFP is a measure of local population neural activity, <a href="https://www.ncbi.nlm.nih.gov/pubmed/22595786">produced from small aggregates of neurons</a>. In these data, we examine the association between rhythms of different frequencies.</p>
<p>In general, lower-frequency rhythms have been observed to engage larger brain areas and modulate spatially localized fast oscillations (see, <a href="https://www.ncbi.nlm.nih.gov/pubmed/18388295">for example</a>). Of the different types of cross-frequency coupling <a href="https://www.ncbi.nlm.nih.gov/pubmed/26549886">(CFC) observced between brain rhythms</a>, perhaps the most is <strong>phase-amplitude coupling</strong> (PAC), in which the phase of a low frequency rhythm modulates the amplitude (or power) of a high frequency rhythm. Cross-frequency coupling been observed in many brain regions, has been shown to change in time with task demands, and has been proposed to serve a <a href="https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3359652/">functional role</a> in working memory, neuronal computation, communication, and learning. Although the cellular and dynamic mechanisms of specific rhythms associated with CFC are relatively well understood, the mechanisms governing interactions between different frequency rhythms - and the appropriate techniques for measuring CFC - remain active research areas. Although we consider only a single electrode recording here, we note that these techniques can be extended to association measures between electrodes as well.</p>
<h3 id="Case-Study-Data">Case Study Data<a class="anchor-link" href="#Case-Study-Data"> </a></h3><p>We are approached by a collaborator recording the local field potential (LFP) from rat hippocampus. She has implanted a small bundle of electrodes, which remain (chronically) implanted as the rat explores a large circular arena. She is interested in assessing the association between different frequency rhythms of the LFP, and more specifically whether an association between different frequency rhythms exists as the rat explores the arena. To address this question, she has provided us with 100 s of LFP data recorded during the experiment (i.e., while the rat spontaneously explored the arena).</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="Goal">Goal<a class="anchor-link" href="#Goal"> </a></h3><p>Our goal is to assess the associations between different frequency rhythms recorded in the LFP. To do so, we analyze the LFP data by computing the phase-amplitude coupling (CFC) of the time series. We construct two CFC measures that characterize how the phase of a low-frequency signal modulates the amplitude envelope of a high-frequency signal.</p>
<h3 id="Tools">Tools<a class="anchor-link" href="#Tools"> </a></h3><p>In this chapter, we develop two CFC measures, with a particular focus on phase-amplitude coupling (PAC). We introduce the concepts of the Hilbert transform, analytic signal, instantaneous phase, and amplitude envelope.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">YouTubeVideo</span><span class="p">(</span><span class="s1">&#39;mLglqyb55_Y&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">


<div class="output_html rendered_html output_subarea output_execute_result">

<iframe
    width="400"
    height="300"
    src="https://www.youtube.com/embed/mLglqyb55_Y"
    frameborder="0"
    allowfullscreen
></iframe>

</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="Data-Analysis">Data Analysis<a id="data-analysis" /><a class="anchor-link" href="#Data-Analysis"> </a></h2><h3 id="Visual-Inspection">Visual Inspection<a class="anchor-link" href="#Visual-Inspection"> </a></h3><p>Let's begin with visual inspection of the LFP data.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="c1"># Load the modules and set plot defaults</span>
<span class="kn">from</span> <span class="nn">scipy.io</span> <span class="kn">import</span> <span class="n">loadmat</span>
<span class="kn">import</span> <span class="nn">matplotlib.pyplot</span> <span class="k">as</span> <span class="nn">plt</span>
<span class="kn">from</span> <span class="nn">matplotlib</span> <span class="kn">import</span> <span class="n">rcParams</span>
<span class="o">%</span><span class="k">matplotlib</span> inline
<span class="n">rcParams</span><span class="p">[</span><span class="s1">&#39;figure.figsize&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="p">(</span><span class="mi">12</span><span class="p">,</span><span class="mi">3</span><span class="p">)</span>
<span class="n">rcParams</span><span class="p">[</span><span class="s1">&#39;axes.xmargin&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="mi">0</span>

<span class="n">data</span> <span class="o">=</span> <span class="n">loadmat</span><span class="p">(</span><span class="s1">&#39;LFP-1.mat&#39;</span><span class="p">)</span>  <span class="c1"># Load the LFP data, </span>
<span class="n">t</span> <span class="o">=</span> <span class="n">data</span><span class="p">[</span><span class="s1">&#39;t&#39;</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span>             <span class="c1"># ... extract t, the time variable,</span>
<span class="n">LFP</span> <span class="o">=</span> <span class="n">data</span><span class="p">[</span><span class="s1">&#39;LFP&#39;</span><span class="p">][</span><span class="mi">0</span><span class="p">]</span>         <span class="c1"># ... and LFP, the voltage variable,</span>
<span class="n">plt</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">t</span><span class="p">,</span> <span class="n">LFP</span><span class="p">)</span>             <span class="c1"># ... and plot the trace,</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlabel</span><span class="p">(</span><span class="s1">&#39;Time [s]&#39;</span><span class="p">)</span>       <span class="c1"># ... with axes labeled.</span>
<span class="n">plt</span><span class="o">.</span><span class="n">ylabel</span><span class="p">(</span><span class="s1">&#39;Voltage [mV]&#39;</span><span class="p">);</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">



<div class="output_png output_subarea ">
<img src="../images/07/cross-frequency-coupling_12_0.png"
>
</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>Next, let's take a closer look at an example 1 s interval of the data<a id="fig:7.1"></a>:</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">plt</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">t</span><span class="p">,</span> <span class="n">LFP</span><span class="p">)</span>            <span class="c1"># Plot the LFP data,</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlim</span><span class="p">([</span><span class="mi">4</span><span class="p">,</span> <span class="mi">5</span><span class="p">])</span>            <span class="c1"># ... restrict the x-axis to a 1 s interval,</span>
<span class="n">plt</span><span class="o">.</span><span class="n">ylim</span><span class="p">([</span><span class="o">-</span><span class="mi">2</span><span class="p">,</span> <span class="mi">2</span><span class="p">])</span>           <span class="c1"># ... and tighten the y-axis.</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlabel</span><span class="p">(</span><span class="s1">&#39;Time [s]&#39;</span><span class="p">)</span>      <span class="c1"># Label the axes</span>
<span class="n">plt</span><span class="o">.</span><span class="n">ylabel</span><span class="p">(</span><span class="s1">&#39;Voltage [mV]&#39;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">show</span><span class="p">()</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">



<div class="output_png output_subarea ">
<img src="../images/07/cross-frequency-coupling_14_0.png"
>
</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>Visual inspection immediately suggests a dominant low-frequency rhythm interspersed with smaller-amplitude blasts of high-frequency activity.</p>
<div class="question">

**Q.** Approximate features of the rhythmic activity by visual inspection of the plot above. What is the frequency of the large-amplitude rhythm? Do you observe high-frequency activity? If so, where in time, and at what approximate frequency? What is the sampling frequency of these data? If you were to compute the spectrum of the entire dataset (100 s of LFP), what would be the <abbr title="The Nyquist frequency is the highest frequency we can possibly hope to observe in the data.">Nyquist frequency</abbr> and the <abbr title="This tells us how fine our estimates of the spectrum will be.">frequency resolution</abbr>? *Hint:* Consider the times near 4.35 s and 4.5 s. Do you see the transient fast oscillations?

</div>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="Spectral-Analysis">Spectral Analysis<a id="spectral-analysis" /><a class="anchor-link" href="#Spectral-Analysis"> </a></h3><p>Visual inspection of the LFP data suggests that multiple rhythms appear. To further characterize this observation, we compute the spectrum of the LFP data. We analyze the entire 100 s of data and compute the spectrum with a Hanning taper.<a id="fig:7.1"></a></p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="kn">import</span> <span class="nn">numpy</span> <span class="k">as</span> <span class="nn">np</span>                   <span class="c1"># Import the NumPy module</span>
<span class="n">dt</span> <span class="o">=</span> <span class="n">t</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span> <span class="o">-</span> <span class="n">t</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span>                     <span class="c1"># Define the sampling interval,</span>
<span class="n">T</span> <span class="o">=</span> <span class="n">t</span><span class="p">[</span><span class="o">-</span><span class="mi">1</span><span class="p">]</span>                            <span class="c1"># ... the duration of the data,</span>
<span class="n">N</span> <span class="o">=</span> <span class="nb">len</span><span class="p">(</span><span class="n">LFP</span><span class="p">)</span>                         <span class="c1"># ... and the no. of data points</span>

<span class="n">x</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">hanning</span><span class="p">(</span><span class="n">N</span><span class="p">)</span> <span class="o">*</span> <span class="n">LFP</span>              <span class="c1"># Multiply data by a Hanning taper</span>
<span class="n">xf</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">fft</span><span class="o">.</span><span class="n">rfft</span><span class="p">(</span><span class="n">x</span> <span class="o">-</span> <span class="n">x</span><span class="o">.</span><span class="n">mean</span><span class="p">())</span>       <span class="c1"># Compute Fourier transform</span>
<span class="n">Sxx</span> <span class="o">=</span> <span class="mi">2</span><span class="o">*</span><span class="n">dt</span><span class="o">**</span><span class="mi">2</span><span class="o">/</span><span class="n">T</span> <span class="o">*</span> <span class="p">(</span><span class="n">xf</span><span class="o">*</span><span class="n">np</span><span class="o">.</span><span class="n">conj</span><span class="p">(</span><span class="n">xf</span><span class="p">))</span>   <span class="c1"># Compute the spectrum</span>
<span class="n">Sxx</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">real</span><span class="p">(</span><span class="n">Sxx</span><span class="p">)</span>                   <span class="c1"># Ignore complex components</span>

<span class="n">df</span> <span class="o">=</span> <span class="mi">1</span> <span class="o">/</span> <span class="n">T</span>                           <span class="c1"># Define frequency resolution,</span>
<span class="n">fNQ</span> <span class="o">=</span> <span class="mi">1</span> <span class="o">/</span> <span class="n">dt</span> <span class="o">/</span> <span class="mi">2</span>                     <span class="c1"># ... and Nyquist frequency. </span>

<span class="n">faxis</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">arange</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="n">fNQ</span> <span class="o">+</span> <span class="n">df</span><span class="p">,</span> <span class="n">df</span><span class="p">)</span>   <span class="c1"># Construct freq. axis</span>
<span class="n">plt</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">faxis</span><span class="p">,</span> <span class="mi">10</span> <span class="o">*</span> <span class="n">np</span><span class="o">.</span><span class="n">log10</span><span class="p">(</span><span class="n">Sxx</span><span class="p">))</span>  <span class="c1"># Plot spectrum vs freq.</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlim</span><span class="p">([</span><span class="mi">0</span><span class="p">,</span> <span class="mi">200</span><span class="p">])</span>                   <span class="c1"># Set freq. range, </span>
<span class="n">plt</span><span class="o">.</span><span class="n">ylim</span><span class="p">([</span><span class="o">-</span><span class="mi">80</span><span class="p">,</span> <span class="mi">0</span><span class="p">])</span>                   <span class="c1"># ... and decibel range</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlabel</span><span class="p">(</span><span class="s1">&#39;Frequency [Hz]&#39;</span><span class="p">)</span>         <span class="c1"># Label the axes</span>
<span class="n">plt</span><span class="o">.</span><span class="n">ylabel</span><span class="p">(</span><span class="s1">&#39;Power [mV$^2$/Hz]&#39;</span><span class="p">);</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">



<div class="output_png output_subarea ">
<img src="../images/07/cross-frequency-coupling_17_0.png"
>
</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>The resulting spectrum reveals two intervals of increased power spectral density. The lowest-frequency peak at 6 Hz is also the largest and corresponds to the slow rhythm we observe dominating the signal through visual inspection. Remember a plot of that signal:
<a href="#fig:7.1" class="fig"><span><img src="imgs/7-1.png"></span></a>
At higher frequencies, we find an additional broadband peak at approximately 80–120 Hz. These spectral results support our initial visual inspection of the signal; there exist both low- and high-frequency activities in the LFP data. We now consider the primary question of interest: Do these different frequency rhythms exhibit associations?</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><a id="cfc"></a></p>
<h3 id="Phase-Amplitude-Coupling">Phase-Amplitude Coupling<a class="anchor-link" href="#Phase-Amplitude-Coupling"> </a></h3>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">YouTubeVideo</span><span class="p">(</span><span class="s1">&#39;JjOcJy4dVzE&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">


<div class="output_html rendered_html output_subarea output_execute_result">

<iframe
    width="400"
    height="300"
    src="https://www.youtube.com/embed/JjOcJy4dVzE"
    frameborder="0"
    allowfullscreen
></iframe>

</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>To assess whether different frequency rhythms interact in the LFP recording, we implement a measure to calculate the CFC. The idea of CFC analysis, in our case, is to determine whether a relation exists between the phase of a low-frequency signal and the envelope or amplitude of a high-frequency signal. In general, computing CFC involves three steps. Each step contains important questions and encompasses entire fields of study. Our goal in this section is to move quickly forward and produce a procedure we can employ, investigate, and criticize. Continued study of CFC - and the associated nuances of each step - is an active area of <a href="https://www.ncbi.nlm.nih.gov/pubmed/26549886">ongoing research</a>.</p>
<h3 id="CFC-analysis-steps">CFC analysis steps<a class="anchor-link" href="#CFC-analysis-steps"> </a></h3><ul>
<li><p>Filter the data into high- and low-frequency bands.</p>
</li>
<li><p>Extract the amplitude and phase from the filtered signals.</p>
</li>
<li><p>Determine if the phase and amplitude are related.</p>
</li>
</ul>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><a id="step1"></a></p>
<h3 id="Step-1.-Filter-the-Data-into-High--and-Low-Frequency-Bands.">Step 1. Filter the Data into High- and Low-Frequency Bands.<a class="anchor-link" href="#Step-1.-Filter-the-Data-into-High--and-Low-Frequency-Bands."> </a></h3>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">YouTubeVideo</span><span class="p">(</span><span class="s1">&#39;WL_nFRBHqLU&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">


<div class="output_html rendered_html output_subarea output_execute_result">

<iframe
    width="400"
    height="300"
    src="https://www.youtube.com/embed/WL_nFRBHqLU"
    frameborder="0"
    allowfullscreen
></iframe>

</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>The first step in the CFC analysis is to filter the data into two frequency bands of interest. The choice is not arbitrary: the separate frequency bands are motivated by initial spectral analysis of the LFP data. In this case, we choose the low-frequency band as 5–7 Hz, consistent with the largest peak in the spectrum, and the high-frequency band as 80–120 Hz, consistent with the second-largest broadband peak. To consider alternative frequency bands, the same analysis steps would apply.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>There are many options to perform the filtering. To do so requires us to design a filter that ideally extracts the frequency bands of interest without distorting the results. Here, we apply a finite impulse response (FIR) filter. We start by extracting the <strong>low-frequency</strong> band:</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="kn">from</span> <span class="nn">scipy</span> <span class="kn">import</span> <span class="n">signal</span>
<span class="n">Wn</span> <span class="o">=</span> <span class="p">[</span><span class="mi">5</span><span class="p">,</span><span class="mi">7</span><span class="p">];</span>                         <span class="c1"># Set the passband [5-7] Hz,</span>
<span class="n">n</span> <span class="o">=</span> <span class="mi">100</span><span class="p">;</span>                            <span class="c1"># ... and filter order,</span>
                                    <span class="c1"># ... build the bandpass filter,</span>
<span class="n">b</span> <span class="o">=</span> <span class="n">signal</span><span class="o">.</span><span class="n">firwin</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">Wn</span><span class="p">,</span> <span class="n">nyq</span><span class="o">=</span><span class="n">fNQ</span><span class="p">,</span> <span class="n">pass_zero</span><span class="o">=</span><span class="kc">False</span><span class="p">,</span> <span class="n">window</span><span class="o">=</span><span class="s1">&#39;hamming&#39;</span><span class="p">);</span>
<span class="n">Vlo</span> <span class="o">=</span> <span class="n">signal</span><span class="o">.</span><span class="n">filtfilt</span><span class="p">(</span><span class="n">b</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="n">LFP</span><span class="p">);</span>   <span class="c1"># ... and apply it to the data.</span>
</pre></div>

    </div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>Next, we extract the <strong>high-frequency</strong> band:</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">Wn</span> <span class="o">=</span> <span class="p">[</span><span class="mi">80</span><span class="p">,</span> <span class="mi">120</span><span class="p">];</span>                     <span class="c1"># Set the passband [80-120] Hz,</span>
<span class="n">n</span> <span class="o">=</span> <span class="mi">100</span><span class="p">;</span>                            <span class="c1"># ... and filter order,</span>
                                    <span class="c1"># ... build the bandpass filter,</span>
<span class="n">b</span> <span class="o">=</span> <span class="n">signal</span><span class="o">.</span><span class="n">firwin</span><span class="p">(</span><span class="n">n</span><span class="p">,</span> <span class="n">Wn</span><span class="p">,</span> <span class="n">nyq</span><span class="o">=</span><span class="n">fNQ</span><span class="p">,</span> <span class="n">pass_zero</span><span class="o">=</span><span class="kc">False</span><span class="p">,</span> <span class="n">window</span><span class="o">=</span><span class="s1">&#39;hamming&#39;</span><span class="p">);</span>
<span class="n">Vhi</span> <span class="o">=</span> <span class="n">signal</span><span class="o">.</span><span class="n">filtfilt</span><span class="p">(</span><span class="n">b</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="n">LFP</span><span class="p">);</span>   <span class="c1"># ... and apply it to the data.</span>
</pre></div>

    </div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>For each frequency band, we specify a frequency interval of interest by defining the low- and high-cutoff frequencies in the variable <code>Wn</code>. In this way, we specify the passband of the filter. We then set the filter order (<code>n</code>) and design the filter using the Python function <code>signal.firwin</code> from the <a href="https://scipy.org/">SciPy package</a>. Finally, we apply the filter using the Python function <code>signal.filtfilt</code>, which performs zero-phase filtering by applying the filter in both the forward and reverse directions.  We note that, in this case, the filtering procedure is nearly the same in both frequency bands; the only change is the specification of the frequency interval of interest.</p>
<p>To understand the impact of this filtering operation on the LFP, let’s plot the results. More specifically, let's plot the original signal, and the signal filtered in the low- and high-frequency bands, for an example 2 s interval of time:<a id="fig:7.3"></a></p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">plt</span><span class="o">.</span><span class="n">figure</span><span class="p">(</span><span class="n">figsize</span><span class="o">=</span><span class="p">(</span><span class="mi">14</span><span class="p">,</span> <span class="mi">4</span><span class="p">))</span>         <span class="c1"># Create a figure with a specific size.</span>
<span class="n">plt</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">t</span><span class="p">,</span> <span class="n">LFP</span><span class="p">)</span>                    <span class="c1"># Plot the original data vs time.</span>
<span class="n">plt</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">t</span><span class="p">,</span> <span class="n">Vlo</span><span class="p">)</span>                    <span class="c1"># Plot the low-frequency filtered data vs time.</span>
<span class="n">plt</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">t</span><span class="p">,</span> <span class="n">Vhi</span><span class="p">)</span>                    <span class="c1"># Plot the high-frequency filtered data vs time.</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlabel</span><span class="p">(</span><span class="s1">&#39;Time [s]&#39;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlim</span><span class="p">([</span><span class="mi">24</span><span class="p">,</span> <span class="mi">26</span><span class="p">]);</span>                 <span class="c1"># Choose a 2 s interval to examine</span>
<span class="n">plt</span><span class="o">.</span><span class="n">ylim</span><span class="p">([</span><span class="o">-</span><span class="mi">2</span><span class="p">,</span> <span class="mi">2</span><span class="p">]);</span>
<span class="n">plt</span><span class="o">.</span><span class="n">legend</span><span class="p">([</span><span class="s1">&#39;LFP&#39;</span><span class="p">,</span> <span class="s1">&#39;Vlo&#39;</span><span class="p">,</span> <span class="s1">&#39;Vhi&#39;</span><span class="p">]);</span>  <span class="c1"># Add a legend.</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">



<div class="output_png output_subarea ">
<img src="../images/07/cross-frequency-coupling_30_0.png"
>
</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>As expected, the low-frequency band (orange) captures the large-amplitude rhythm dominating the LFP signal, while the higher-frequency band (green) isolates the brief bursts of faster activity.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><a id="step2"></a></p>
<h3 id="Step-2.-Extract-the-Amplitude-and-Phase-from-Filtered-Signals.">Step 2. Extract the Amplitude and Phase from Filtered Signals.<a class="anchor-link" href="#Step-2.-Extract-the-Amplitude-and-Phase-from-Filtered-Signals."> </a></h3>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">YouTubeVideo</span><span class="p">(</span><span class="s1">&#39;QiRuBvbCQt4&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">


<div class="output_html rendered_html output_subarea output_execute_result">

<iframe
    width="400"
    height="300"
    src="https://www.youtube.com/embed/QiRuBvbCQt4"
    frameborder="0"
    allowfullscreen
></iframe>

</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>The next step in the CFC procedure is to extract the <em>phase</em> of the low-frequency signal and the amplitude envelope (or simply, <em>amplitude</em>) of the high-frequency signal. To compute a particular type of CFC, namely phase-amplitude coupling, we then compare these two signals, i.e., we compare the phase of the low-frequency activity and the amplitude envelope of the high-frequency activity. How do we actually extract the phase and amplitude signals from the data? There are a variety of options to do so, and we choose here to employ the <strong>analytic signal approach</strong>, which allows us to estimate the instantaneous phase and amplitude envelope of the LFP.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">YouTubeVideo</span><span class="p">(</span><span class="s1">&#39;Ir8Gf550S3o&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">


<div class="output_html rendered_html output_subarea output_execute_result">

<iframe
    width="400"
    height="300"
    src="https://www.youtube.com/embed/Ir8Gf550S3o"
    frameborder="0"
    allowfullscreen
></iframe>

</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>The first step in computing the analytic signal is to compute the <a href="https://en.wikipedia.org/wiki/Hilbert_transform">Hilbert transform</a>. We begin with some notation. Define $x$ as a narrowband signal (i.e., a signal with most of its energy concentrated in a narrow frequency range<sup> <abbr title="The impact of this narrowband assumption on CFC estimates remains an open research topic. One might consider, for example, the meaning and implications of estimating phase from a broadband signal, and the impact on subsequent CFC results.">note</abbr></sup>, e.g., the low- or high-frequency band filtered signals we've created). Then the Hilbert transform of $x$, let’s call it $y$, is</p>
$$y = H(x).$$<p>It’s perhaps more intuitive to consider the effect of the Hilbert Transform on the frequencies
$f$ of $x$,</p>
$$
\begin{equation}
    H(x)=
    \begin{cases}
      -\pi/2 &amp; \text{ phase shift if } f&gt;0 \\
      0 &amp; \text{ phase shift if } f=0 \\
      \pi/2 &amp; \text{ phase shift if } f&lt;0 \\
    \end{cases}
  \end{equation}
$$<div class="math-note">

**To summarize**:  The Hilbert transform $H(x)$ of the signal $x$ produces a phase shift of $\pm 90$ degrees for $\mp$ frequencies of $x$.

</div><p>The Hilbert Transform can also be described in the time domain, although its representation is hardly intuitive (see this <a href="#appendix">Appendix</a> for more details). Then the analytic signal $z$ is</p>
$$ z = x + i y = x + i H(x). $$<p>The effect of the Hilbert transform is to remove negative frequencies from $z$. As it stands,
this is not obvious. To get a sense for why this is so, let’s consider a simple example.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><a id="hilbert"></a></p>
<h3 id="What-Does-the-Hilbert-Transform-Do?">What Does the Hilbert Transform Do?<a class="anchor-link" href="#What-Does-the-Hilbert-Transform-Do?"> </a></h3><p><a href="#step2ctd">Skip this section</a></p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">YouTubeVideo</span><span class="p">(</span><span class="s1">&#39;-CjnFEOopfw&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">


<div class="output_html rendered_html output_subarea output_execute_result">

<iframe
    width="400"
    height="300"
    src="https://www.youtube.com/embed/-CjnFEOopfw"
    frameborder="0"
    allowfullscreen
></iframe>

</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>Let $x_0$ be a sinusoid at frequency $f_0$,</p>
$$ x_0 = 2 \cos(2 \pi f_0 t) = 2 \cos(\omega_0 t),$$<p>where to simplify notation we have defined $\omega_0 = 2 \pi f_0$. We know from <a href="https://en.wikipedia.org/wiki/Euler%27s_formula">Euler’s formula</a> that
<a id="eq:7.3"></a></p>
$$x_0 = e^{i \omega_0 t} + e^{-i \omega_0 t}.$$<p>The real variable $x_0$ possesses both a positive and a negative frequency component (i.e., $\omega_0$ and $-\omega_0$). So, the spectrum has two peaks:</p>
<p><img src="imgs/7-5.png" alt="Drawing" style="width: 500px;"/></p>
<p>For real signals, which include nearly all recordings of brain activity, the negative frequency component is redundant, and we usually ignore it. However, the negative frequency component still remains. By definition, the effect of the Hilbert transform is to induce a phase shift. For positive frequencies, the phase shift is  $-\pi/2$. We can produce this phase shift by multiplying the positive frequency part of the signal by $-i$.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<div class="question">

**Q:** Why does a phase shift of  $-\pi/2$ correspond to multiplication by $-i$?

**A:** Consider the complex exponential $e^{i \omega_0 t}$, which consists of only a positive frequency component ($\omega_0$). This signal shifted in phase by $-\pi/2$ corresponds to the new signal $e^{i (\omega_0 t - \pi / 2)}$, which simplifies to

$$e^{i \omega_0 t} e^{-i\pi/2} = e^{i\omega_0 t}\big(\cos(\pi/2) - i \sin(\pi/2)\big) = e^{i\omega_0 t} (-i).$$

Notice the result simplifies to the original complex exponential $e^{i\omega_0 t}$ multiplied by $-i$. This shows that the $-\pi/2$ phase shift corresponds to multiplication of the positive frequency component $(e^{i\omega_0 t})$ by $-i$.

</div>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<div class="question">

**Q.** Can you show that a $\pi / 2$ phase shift corresponds to multiplication by $i$? 

</div>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>This analysis shows that we can represent the Hilbert transform of $x$ at frequency $f$ as</p>
$$ H(x) = \left\{\begin{array}{r}
-ix \mbox{ if } f &gt; 0, \\
0 \mbox{ if } f = 0, \\
ix \mbox{ if } f &lt; 0.
\end{array}\right.$$<p>Therefore, the Hilbert transform of $x_0 = e ^ {i \omega_0 t} + e ^ {-i \omega_0 t}$ becomes</p>
$$y_0 = H(x_0) = -i e ^ {i \omega_0 t} + i e ^ {-i \omega_0 t}. $$<p>In this equation, we multiply the positive frequency part of $x_0$ (i.e., $e ^ {-i \omega_0 t}$) by $i$. Simplifying this expression using Euler's formula, we find</p>
$$ y_0 = 2 \sin( \omega_0 t ).$$<div class="question">

**Q.** Can you perform this simplification? In other words, can you show that $-i e ^ {i \omega_0 t} + i e ^ {-i \omega_0 t} = 2 \sin( \omega_0 t )$? 

</div>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">YouTubeVideo</span><span class="p">(</span><span class="s1">&#39;e4kECy8KP-4&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">


<div class="output_html rendered_html output_subarea output_execute_result">

<iframe
    width="400"
    height="300"
    src="https://www.youtube.com/embed/e4kECy8KP-4"
    frameborder="0"
    allowfullscreen
></iframe>

</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>The Hilbert transform of $x_0$ (a cosine function) is a sine function. We could perhaps have guessed this: sine is a 90-degree ($\pi / 2$) phase shift of cosine.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">YouTubeVideo</span><span class="p">(</span><span class="s1">&#39;emsU97RcFjI&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">


<div class="output_html rendered_html output_subarea output_execute_result">

<iframe
    width="400"
    height="300"
    src="https://www.youtube.com/embed/emsU97RcFjI"
    frameborder="0"
    allowfullscreen
></iframe>

</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>We are now ready to define the analytic signal $(z_0)$ for this example. Using the expressions for $x_0$ and $y_0$ and Euler's formula, we find</p>
$$z_0 = x_0 + i y_0 = 2 \cos( \omega_0 t ) + i 2 \sin(\omega_0 t ) = 2 e ^ {i \omega_0 t}.$$<p>Notice that this analytic signal $z_0$ contains no negative frequencies; as mentioned, the effect of the Hilbert transform is to eliminate the negative frequencies from $x$. The spectrum of this signal consists of a single peak at $\omega_0$, compared to the two peaks at $\pm \omega_0$ in the original signal $x$. In this sense, the analytic signal ($z_0$) is simpler than the original signal $x_0$. To express the original signal $x_0$ required two complex exponential functions,</p>
$$x_0 = e^{i \omega_0 t} + e^{-i \omega_0 t},$$<p>compared to only one complex exponential required to express the corresponding analytic signal $z_0$. There's an interesting geometrical interpretation of this. Consider plotting $x_0$ in the complex plane:</p>
<p><img id="fig:7.6" src="imgs/7-6.png" style="width:30vw"></img></p>
<p>Because $x_0$ is real, this quantity evolves in time along the real axis (red in the figure). To keep $x_0$ on the real axis, the two complex exponentials (orange in the figure) that define $x_0$ (i.e., $e^{i \omega_0 t}$ and $e^{-i \omega_0 t}$) rotate in opposite directions along the unit circle (blue in the figure). By doing so, the imaginary components of these two vectors cancel, and we're left with a purely real quantity $x_0$.</p>
<div class="question">

**Q.** The phase of a complex quantity is the angle with respect to the real axis in the complex plain. What is the angle of $x_0$ in the figure above? 

</div>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><a id="step2ctd"></a></p>
<h3 id="Step-2.-Extract-the-Amplitude-and-Phase-from-Filtered-Signals-(Continued).">Step 2. Extract the Amplitude and Phase from Filtered Signals (Continued).<a class="anchor-link" href="#Step-2.-Extract-the-Amplitude-and-Phase-from-Filtered-Signals-(Continued)."> </a></h3><p>Having developed some understanding of the Hilbert Transform, let’s now return to the LFP data of interest here. It’s relatively easy to compute the analytic signal and extract the phase and amplitude in Python:</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">phi</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">angle</span><span class="p">(</span><span class="n">signal</span><span class="o">.</span><span class="n">hilbert</span><span class="p">(</span><span class="n">Vlo</span><span class="p">))</span>  <span class="c1"># Compute phase of low-freq signal</span>
<span class="n">amp</span> <span class="o">=</span> <span class="nb">abs</span><span class="p">(</span><span class="n">signal</span><span class="o">.</span><span class="n">hilbert</span><span class="p">(</span><span class="n">Vhi</span><span class="p">))</span>       <span class="c1"># Compute amplitude of high-freq signal</span>
</pre></div>

    </div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>These operations require just two lines of code. But beware the following.</p>
<div class="python-note">

**Alert!** The command `hilbert(x)` returns the analytic signal of $x$, not the Hilbert transform of $x$.

</div><p>To extract the phase, we apply the function <code>angle</code> to the analytic signal of the data filtered in the low-frequency band (variable <code>Vlo</code>). We then extract the amplitude of the analytic signal of the data filtered in the high-frequency band (variable <code>Vhi</code>) by computing the absolute value (function <code>abs</code>).</p>
<p>To summarize, in this step we apply the Hilbert transform to create the analytic signal and get the phase or amplitude of the bandpass-filtered data.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><a id="step3"></a></p>
<h3 id="Step-3.-Determine-if-the-Phase-and-Amplitude-are-Related.">Step 3. Determine if the Phase and Amplitude are Related.<a class="anchor-link" href="#Step-3.-Determine-if-the-Phase-and-Amplitude-are-Related."> </a></h3>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">YouTubeVideo</span><span class="p">(</span><span class="s1">&#39;fiL9UNbLPA8&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">


<div class="output_html rendered_html output_subarea output_execute_result">

<iframe
    width="400"
    height="300"
    src="https://www.youtube.com/embed/fiL9UNbLPA8"
    frameborder="0"
    allowfullscreen
></iframe>

</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>As with the previous steps, we have at our disposal a variety of procedures to assess the relation between the phase (of the low-frequency signal) and amplitude (of the high-frequency signal). We do so here in one way, by computing the phase-amplitude plot.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><a id="m1"></a></p>
<h3 id="The-phase-amplitude-plot">The phase-amplitude plot<a class="anchor-link" href="#The-phase-amplitude-plot"> </a></h3><p>To start, define the two-column phase-amplitude vector,</p>
$$\left( \begin{array}{cc}
\phi(1) &amp; A(1) \\
\phi(2) &amp; A(2) \\
\phi(3) &amp; A(3) \\
\vdots &amp; \vdots \\
\end{array}\right),$$<p>where $\phi(i)$ is the phase of the low-frequency band activity at time index $i$, and $A(i)$ is the amplitude of the high-frequency band activity at time index $i$. In other words, each row defines the instantaneous phase and amplitude of the low- and high-frequency filtered data, respectively.</p>
<p>We now use this two-column vector to assess whether the phase and amplitude envelope are related. Let's begin by plotting the average amplitude versus phase. We divide the phase interval into bins of size 0.1 beginning at $-\pi$ and ending at $\pi.$ the choice of bin size is somewhat arbitrary; this choice will work fine, but you might consider the impact of other choices.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">p_bins</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">arange</span><span class="p">(</span><span class="o">-</span><span class="n">np</span><span class="o">.</span><span class="n">pi</span><span class="p">,</span> <span class="n">np</span><span class="o">.</span><span class="n">pi</span><span class="p">,</span> <span class="mf">0.1</span><span class="p">)</span>
<span class="n">a_mean</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">zeros</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">size</span><span class="p">(</span><span class="n">p_bins</span><span class="p">)</span><span class="o">-</span><span class="mi">1</span><span class="p">)</span>
<span class="n">p_mean</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">zeros</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">size</span><span class="p">(</span><span class="n">p_bins</span><span class="p">)</span><span class="o">-</span><span class="mi">1</span><span class="p">)</span>
<span class="k">for</span> <span class="n">k</span> <span class="ow">in</span> <span class="nb">range</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">size</span><span class="p">(</span><span class="n">p_bins</span><span class="p">)</span><span class="o">-</span><span class="mi">1</span><span class="p">):</span>
    <span class="n">pL</span> <span class="o">=</span> <span class="n">p_bins</span><span class="p">[</span><span class="n">k</span><span class="p">]</span>					    <span class="c1">#... lower phase limit,</span>
    <span class="n">pR</span> <span class="o">=</span> <span class="n">p_bins</span><span class="p">[</span><span class="n">k</span><span class="o">+</span><span class="mi">1</span><span class="p">]</span>				    <span class="c1">#... upper phase limit.</span>
    <span class="n">indices</span><span class="o">=</span><span class="p">(</span><span class="n">phi</span><span class="o">&gt;=</span><span class="n">pL</span><span class="p">)</span> <span class="o">&amp;</span> <span class="p">(</span><span class="n">phi</span><span class="o">&lt;</span><span class="n">pR</span><span class="p">)</span>	    <span class="c1">#Find phases falling in bin,</span>
    <span class="n">a_mean</span><span class="p">[</span><span class="n">k</span><span class="p">]</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">mean</span><span class="p">(</span><span class="n">amp</span><span class="p">[</span><span class="n">indices</span><span class="p">])</span>	<span class="c1">#... compute mean amplitude,</span>
    <span class="n">p_mean</span><span class="p">[</span><span class="n">k</span><span class="p">]</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">mean</span><span class="p">([</span><span class="n">pL</span><span class="p">,</span> <span class="n">pR</span><span class="p">])</span>		<span class="c1">#... save center phase.</span>
<span class="n">plt</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">p_mean</span><span class="p">,</span> <span class="n">a_mean</span><span class="p">)</span>                <span class="c1">#Plot the phase versus amplitude,</span>
<span class="n">plt</span><span class="o">.</span><span class="n">ylabel</span><span class="p">(</span><span class="s1">&#39;High-frequency amplitude&#39;</span><span class="p">)</span>  <span class="c1">#... with axes labeled.</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlabel</span><span class="p">(</span><span class="s1">&#39;Low-frequency phase&#39;</span><span class="p">);</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">



<div class="output_png output_subarea ">
<img src="../images/07/cross-frequency-coupling_54_0.png"
>
</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<div class="question">

**Q.** Consider this plot of the average amplitude versus phase. Does this result suggest CFC occurs in these data?

**A.** The plot shows the phase bins versus the mean amplitude in each bin. Visual inspection of this phase-amplitude plot suggests that the amplitude of the high-frequency signal depends on the phase of the low-frequency signal. In particular, we note that when the phase is near a value of 2 radians, the amplitude tends to be large, while at other phases the amplitude is smaller. This conclusion suggests that CFC does occur in the data; the high-frequency amplitude depends on the low-frequency phase.

</div>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<div class="question">

**Q.** If no CFC occurred in the data, what would you expect to find in the plot of average amplitude versus phase? 

</div>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>As a basic statistic to capture the extent of this relation, we compute the difference between the maximum and minimum of the average amplitude envelope over phases. Let's assign this difference the label $h$. In Python,</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">h</span> <span class="o">=</span> <span class="nb">max</span><span class="p">(</span><span class="n">a_mean</span><span class="p">)</span><span class="o">-</span><span class="nb">min</span><span class="p">(</span><span class="n">a_mean</span><span class="p">)</span>
<span class="nb">print</span><span class="p">(</span><span class="n">h</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">

<div class="output_subarea output_stream output_stdout output_text">
<pre>0.12607449865513892
</pre>
</div>
</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>We find a value of $h = 0.126$. This value, on its own, is difficult to interpret. Is it bigger or smaller than we expect? To assess the significance of $h$, let's generate a surrogate phase-amplitude vector by resampling without replacement the amplitude time series (i.e., the second column of the phase-amplitude vector). Resampling is a powerful technique that we have applied in our analysis of other case study data (see, for example, 
<a href="../The%20Event-Related%20Potential/The%20Event-Related%20Potential.ipynb">The Event-Related Potential</a>). By performing this resampling, we reassign each phase an amplitude chosen randomly from the entire 100 s LFP recording. We expect that if CFC does exist in these data, then the timing of the phase and amplitude vectors will be important; for CFC to occur, the amplitude and phase must coordinate in time. By disrupting this timing in the resampling procedure, we expect to eliminate the coordination between amplitude and phase necessary to produce CFC.</p>
<p>For each surrogate phase-amplitude vector, we compute the statistic $h$. To generate a distribution of $h$ values, let’s repeat 1,000 times this process of creating surrogate data through resampling and computing $h$.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">n_surrogates</span> <span class="o">=</span> <span class="mi">1000</span><span class="p">;</span>                       <span class="c1">#Define no. of surrogates.</span>
<span class="n">hS</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">zeros</span><span class="p">(</span><span class="n">n_surrogates</span><span class="p">)</span>                <span class="c1">#Vector to hold h results.</span>
<span class="k">for</span> <span class="n">ns</span> <span class="ow">in</span> <span class="nb">range</span><span class="p">(</span><span class="n">n_surrogates</span><span class="p">):</span>             <span class="c1">#For each surrogate,</span>
    <span class="n">ampS</span> <span class="o">=</span> <span class="n">amp</span><span class="p">[</span><span class="n">np</span><span class="o">.</span><span class="n">random</span><span class="o">.</span><span class="n">randint</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span><span class="n">N</span><span class="p">,</span><span class="n">N</span><span class="p">)]</span>   <span class="c1">#Resample amplitude,</span>
    <span class="n">p_bins</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">arange</span><span class="p">(</span><span class="o">-</span><span class="n">np</span><span class="o">.</span><span class="n">pi</span><span class="p">,</span> <span class="n">np</span><span class="o">.</span><span class="n">pi</span><span class="p">,</span> <span class="mf">0.1</span><span class="p">)</span> <span class="c1">#Define the phase bins</span>
    <span class="n">a_mean</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">zeros</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">size</span><span class="p">(</span><span class="n">p_bins</span><span class="p">)</span><span class="o">-</span><span class="mi">1</span><span class="p">)</span>   <span class="c1">#Vector for average amps.</span>
    <span class="n">p_mean</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">zeros</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">size</span><span class="p">(</span><span class="n">p_bins</span><span class="p">)</span><span class="o">-</span><span class="mi">1</span><span class="p">)</span>   <span class="c1">#Vector for phase bins.</span>
    <span class="k">for</span> <span class="n">k</span> <span class="ow">in</span> <span class="nb">range</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">size</span><span class="p">(</span><span class="n">p_bins</span><span class="p">)</span><span class="o">-</span><span class="mi">1</span><span class="p">):</span>
        <span class="n">pL</span> <span class="o">=</span> <span class="n">p_bins</span><span class="p">[</span><span class="n">k</span><span class="p">]</span>					    <span class="c1">#... lower phase limit,</span>
        <span class="n">pR</span> <span class="o">=</span> <span class="n">p_bins</span><span class="p">[</span><span class="n">k</span><span class="o">+</span><span class="mi">1</span><span class="p">]</span>				    <span class="c1">#... upper phase limit.</span>
        <span class="n">indices</span><span class="o">=</span><span class="p">(</span><span class="n">phi</span><span class="o">&gt;=</span><span class="n">pL</span><span class="p">)</span> <span class="o">&amp;</span> <span class="p">(</span><span class="n">phi</span><span class="o">&lt;</span><span class="n">pR</span><span class="p">)</span>	    <span class="c1">#Find phases falling in bin,</span>
        <span class="n">a_mean</span><span class="p">[</span><span class="n">k</span><span class="p">]</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">mean</span><span class="p">(</span><span class="n">ampS</span><span class="p">[</span><span class="n">indices</span><span class="p">])</span>	<span class="c1">#... compute mean amplitude,</span>
        <span class="n">p_mean</span><span class="p">[</span><span class="n">k</span><span class="p">]</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">mean</span><span class="p">([</span><span class="n">pL</span><span class="p">,</span> <span class="n">pR</span><span class="p">])</span>		<span class="c1">#... save center phase.</span>
    <span class="n">hS</span><span class="p">[</span><span class="n">ns</span><span class="p">]</span> <span class="o">=</span> <span class="nb">max</span><span class="p">(</span><span class="n">a_mean</span><span class="p">)</span><span class="o">-</span><span class="nb">min</span><span class="p">(</span><span class="n">a_mean</span><span class="p">)</span>        <span class="c1"># Store surrogate h.</span>
</pre></div>

    </div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>In this code, we first define the number of surrogates (variable <code>n_surrogates</code>) and a vector to store the statistic $h$ computed for each surrogate phase-amplitude vector (variable <code>hS</code>). Then, for each surrogate, we use the Python function <code>random.randint</code> to randomly permute the amplitude vector without replacement. We then use this permuted amplitude vector (variable <code>ampS</code>) and the original phase vector (variable <code>phi</code>) to compute $h$; this last step utilizes the Python code developed earlier to compute <code>h</code> for the original (unpermuted) data.</p>
<p>Let's now view the results of this resampling procedure by creating a histogram of the variable <code>hS</code>, and compare this distribution to the value of <code>h</code> we computed earlier.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">counts</span><span class="p">,</span> <span class="n">_</span><span class="p">,</span> <span class="n">_</span> <span class="o">=</span> <span class="n">plt</span><span class="o">.</span><span class="n">hist</span><span class="p">(</span><span class="n">hS</span><span class="p">,</span> <span class="n">label</span><span class="o">=</span><span class="s1">&#39;surrogates&#39;</span><span class="p">)</span>               <span class="c1"># Plot the histogram of hS, and save the bin counts.</span>
<span class="n">plt</span><span class="o">.</span><span class="n">vlines</span><span class="p">(</span><span class="n">h</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">max</span><span class="p">(</span><span class="n">counts</span><span class="p">),</span> <span class="n">colors</span><span class="o">=</span><span class="s1">&#39;red&#39;</span><span class="p">,</span> <span class="n">label</span><span class="o">=</span><span class="s1">&#39;h&#39;</span><span class="p">,</span> <span class="n">lw</span><span class="o">=</span><span class="mi">2</span><span class="p">)</span>  <span class="c1"># Plot the observed h,</span>
<span class="n">plt</span><span class="o">.</span><span class="n">legend</span><span class="p">();</span>                                                 <span class="c1"># ... include a legend.</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">



<div class="output_png output_subarea ">
<img src="../images/07/cross-frequency-coupling_62_0.png"
>
</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>The value of $h$ computed from the original data (<code>h</code>) lies far outside the distribution of surrogate $h$ values (<code>hS</code>). To compute a $p$-value, we determine the proportion of surrogate $h$ values greater than the observed $h$ value:</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="n">p</span> <span class="o">=</span> <span class="nb">sum</span><span class="p">([</span><span class="n">s</span> <span class="o">&gt;</span> <span class="n">h</span> <span class="k">for</span> <span class="n">s</span> <span class="ow">in</span> <span class="n">hS</span><span class="p">])</span> <span class="o">/</span> <span class="nb">len</span><span class="p">(</span><span class="n">hS</span><span class="p">)</span>
<span class="nb">print</span><span class="p">(</span><span class="n">p</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">

<div class="output_subarea output_stream output_stdout output_text">
<pre>0.0
</pre>
</div>
</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>We find a $p$-value that is very small; there are no surrogate values of $h$ that exceed the observed value of $h$. We therefore conclude that in this case the observed value of $h$ is significant. As a statistician would say, we reject the null hypothesis of <em>no</em> CFC between the phase of the 5-7 Hz low-frequency band and the amplitude of the 80-120 Hz high frequency band.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="kn">from</span> <span class="nn">IPython.lib.display</span> <span class="kn">import</span> <span class="n">YouTubeVideo</span>
<span class="n">YouTubeVideo</span><span class="p">(</span><span class="s1">&#39;NQUfELSZ0Cc&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">


<div class="output_html rendered_html output_subarea output_execute_result">

<iframe
    width="400"
    height="300"
    src="https://www.youtube.com/embed/NQUfELSZ0Cc"
    frameborder="0"
    allowfullscreen
></iframe>

</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><a id="summary"></a></p>
<h1 id="Summary">Summary<a class="anchor-link" href="#Summary"> </a></h1>
</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="kn">from</span> <span class="nn">IPython.lib.display</span> <span class="kn">import</span> <span class="n">YouTubeVideo</span>
<span class="n">YouTubeVideo</span><span class="p">(</span><span class="s1">&#39;NQUfELSZ0Cc&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

<div class="output_wrapper">
<div class="output">

<div class="jb_output_wrapper }}">
<div class="output_area">


<div class="output_html rendered_html output_subarea output_execute_result">

<iframe
    width="400"
    height="300"
    src="https://www.youtube.com/embed/NQUfELSZ0Cc"
    frameborder="0"
    allowfullscreen
></iframe>

</div>

</div>
</div>
</div>
</div>

</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>In this module, we considered techniques to characterize cross-frequency coupling (CFC), associations between rhythmic activity observed in two different frequency bands. To do so, we introduced the Hilbert transform, which can be used to compute the instantaneous phase and amplitude of a signal. We focused on characterizing the relation between the phase of low-frequency band activity (5-7 Hz) and the amplitude of high-frequency band activity (100-140 Hz). To do this, we computed the average amplitude at each phase and determined the extent of the variability (or wiggliness).</p>
<p>For the LFP data of interest here, we found evidence of CFC between the two frequency bands. Importantly, these results also appear consistent with visual inspection of the unfiltered data. Careful inspection of the example voltage traces suggests that CFC does in fact occur in these data. In general, such strong CFC, visible to the naked eye in the unprocessed LFP data, is unusual. Instead, data analysis techniques are required to detect features not obvious through visual inspection alone. Developing techniques to assess CFC and understanding the biophysical mechanisms of CFC and implications for function, remain active research areas.</p>
<p>In developing these approaches, we utilized expertise and procedures developed in other modules. In particular, we relied on the notions of frequency and power, amplitude and phase, filtering, and resampling. Such a multifaceted approach is typical in analysis of neural data, where we leverage the skills developed in analyzing other datasets.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><a id="appendix"></a></p>
<h2 id="Appendix:-Hilbert-Transform-in-the-Time-Domain">Appendix: Hilbert Transform in the Time Domain<a class="anchor-link" href="#Appendix:-Hilbert-Transform-in-the-Time-Domain"> </a></h2><p>We have presented the Hilbert Transform in the frequency domain: it produces a quarter cycle phase shift. It's reasonable to consider as well the <em>time domain</em> representation of the Hilbert Transform. To do so, let's write the Hilbert Transform as</p>
$$H(x) = \left\{ \begin{array}{rl}
-ix &amp; \mbox{for positive frequencies of }x \\
0 &amp; \mbox{for 0 frequency of }x \\
ix &amp; \mbox{for negative frequencies of }x \\
\end{array}\right\} = x(f)\big( -i\mbox{sgn}(f)\big),$$<p>where we have written $x(f)$ to make the frequency dependence of $x$ explicit, and the sgn (pronounced "sign") function is defined as</p>
$$\mbox{sgn}(f) = \left\{ \begin{array}{rl}
1 &amp; \mbox{if }f &gt; 0, \\
0 &amp; \mbox{if }f = 0, \\
-1 &amp; \mbox{if }f &lt; 0, \\
\end{array}\right.$$<p>In the frequency domain, we perform the Hilbert Transform by multiplying the signal $x(f)$ by a constant (either $i$ or $-i$ depending on the frequency $f$ of $x$).</p>
<p>To convert the Hilbert transform in the frequency domain to the Hilbert Transform in the time domain, we take the inverse Fourier transform. Looking up the inverse Fourier transform of $-i\mbox{sgn}(f)$, we find</p>
$$\mbox{Inverse Fourier transform}\{-i\mbox{sgn}(f)\} = \frac{1}{\pi t}.$$<p>Let's represent the inverse Fourier transform of $x(f)$ as $x(t)$.</p>
<p>Now, let's make use of an important fact. Multiplication of two quantities in the frequency domain corresponds to convolution of these two quantities in the time domain (see <a href="../Analysis%20of%20Rhythmic%20Activity%20in%20an%20Electrocorticogram/Analysis%20of%20rhythmic%20activity%20in%20the%20Electrocorticogram.ipynb">Analysis of Rhythmic Activity in an Electrocorticogram</a>). The convolution of two signals $x$ and $y$ is</p>
$$ x * y = \int_{-\infty}^{\infty}x(\tau)y(t-\tau)d\tau.$$<p>So, in the time domain, the Hilbert transform becomes</p>
$$H(x) = x(t) * \frac{1}{\pi t} = \frac{1}{\pi}\int_{-\infty}^{\infty}\frac{x(\tau)}{t-\tau}d\tau.$$<p>This time domain representation of the HIlbert Transform is equivalent to the frequency domain representation. However, the time domain representation is much less intuitive. Compare the previous equation to the statement, "<em>The Hilbert Transform is a 90-degree phase shift in the frequency domain.</em>" The latter, we propose, is much more intuitive.</p>

</div>
</div>
</div>
</div>

<div class="jb_cell">

<div class="cell border-box-sizing code_cell rendered">
<div class="input">

<div class="inner_cell">
    <div class="input_area">
<div class=" highlight hl-ipython3"><pre><span></span><span class="kn">from</span> <span class="nn">IPython.core.display</span> <span class="kn">import</span> <span class="n">HTML</span>
<span class="n">HTML</span><span class="p">(</span><span class="s1">&#39;../../assets/custom/custom.css&#39;</span><span class="p">)</span>
</pre></div>

    </div>
</div>
</div>

</div>
</div>

 


    </main>
    