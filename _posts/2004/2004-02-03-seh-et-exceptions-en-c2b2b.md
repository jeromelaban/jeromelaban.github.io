---
layout: post
title: "SEH et Exceptions en C++"
date: 2004-02-03 11:49:00 -0500
comments: true
category: Archive
tags: ["Misc"]
redirect_from: ["/post/2004/02/03/SEH-et-Exceptions-en-C2b2b.aspx", "/post/2004/02/03/seh-et-exceptions-en-c2b2b.aspx"]
author: jerome
---
<!-- more -->
<font face="Tahoma"></font>
<p>
<font size="2">Qui ne s&#39;est pas d&eacute;ja retrouv&eacute; devant une horrible boite de dialogue de crash de programme ressemblant &agrave; celle ci :<br />
</font><font size="2"><br />
</font><font size="2"><font face="Courier New">&quot;The memory cannot be &#39;written&#39; at 0x404459FF&quot;.</font></font>
</p>
<p>
<font size="2">Cette boite de dialogue, bien connue des utilisateurs et d&eacute;veloppeurs&nbsp;sur Windows, est le r&eacute;sultat d&#39;une exception&nbsp;mat&eacute;rielle, souvent la cons&eacute;quence d&#39;un probl&egrave;me logiciel comme un der&eacute;f&eacute;rencement de pointeur nul. </font><font size="2">Il existe bien entendu&nbsp;beaucoup raisons pour lesquelles cette boite peut apparaitre comme des <em>Access Violation</em>, <em>Divide By Zero</em>, <em>Invalid Operation</em>, <em>Float Overflow</em> et <em>Underflow</em>, <em>Privileged Instruction</em>,<em>...</em> autant de probl&egrave;mes&nbsp;potentiels qui peuvent arr&ecirc;ter l&#39;ex&eacute;cution d&#39;un programme.&nbsp; </font>
</p>
<p>
<font size="2">Le C++ met &agrave; disposition un m&eacute;canisme de gestion des exceptions par l&#39;interm&eacute;diaire des mots cl&eacute;s <font face="Courier New">try</font> et <font face="Courier New">catch</font> tout en utilisant le handler par d&eacute;faut. Ce handler (<font face="Courier New">catch(...)</font>) du m&eacute;canisme standard&nbsp;n&#39;est cependant pas le plus int&eacute;ressant car il ne permet pas d&#39;obtenir le contexte d&#39;ex&eacute;cution&nbsp;du processeur lors de la g&eacute;n&eacute;ration de l&#39;exception, ni d&#39;en savoir la cause. </font>
</p>
<p>
<font size="2">Windows met &agrave; disposition du d&eacute;veloppeur une API sp&eacute;cifique permettant d&#39;utiliser le <em>Structured Exception Handling</em>. Cela permet, entre autres, d&#39;afficher une boite de dialogue &agrave; l&#39;utilisateur avec un minimum d&#39;informations de debug. Il est &eacute;galement possible d&#39;avoir un StackTrace pour un ex&eacute;cutable en utilisant les informations de debug s&eacute;par&eacute;es. Contrairement &agrave; une id&eacute;e recue, il est possible de g&eacute;nerer avec visual studio des informations de debug pour un ex&eacute;cutable optimis&eacute; avec /Ox par exemple. </font>
</p>
<p>
<font size="2">Le compilateur C++ met &agrave; disposition un certain nombre de nouveaux mots cl&eacute;s permettant de prot&eacute;ger une section de code ou&nbsp;bien un&nbsp;programme entier&nbsp;et d&#39;en intercepter les exceptions d&#39;une mani&egrave;re plus efficace. On peut notamment utiliser __try, __except et __finally qui&nbsp;sp&eacute;cifiques au compilateur Microsoft. Cette m&eacute;thode n&#39;est cependant pas la plus simple &agrave; utiliser. </font>
</p>
<p>
<font size="2">L&#39;utilisation du SEH ici est coupl&eacute;e &agrave; l&#39;utilisation de la librairie imagehlp.dll. Cette librairie permet d&#39;explorer les fichiers de symboles (pdb) pour d&eacute;terminer par exemple les fonctions trouv&eacute;es en parcourant la StackFrame. </font>
</p>
<p>
<font size="2">Cet&nbsp;<a href="http://www.microsoft.com/msj/0497/hood/hood0497.aspx">article du MSJournal de 1997</a>&nbsp;ainsi que <a href="http://www.microsoft.com/msj/0197/Exception/Exception.aspx">celui ci</a>, <em>(Oui, oui, 1997...)</em>&nbsp;d&eacute;crit assez bien le fonctionnement et l&#39;utilisation du SEH au travers de l&#39;utilisation d&#39;une instance de&nbsp;classe en singleton, mais ce mode ne permet pas de rendre simplement la main&nbsp;au dernier handler d&#39;exception pr&eacute;sent. </font>
</p>
<p>
<font size="2">Dans <a href="http://etudiants.epita.fr/~laban_j/blog/cpp-seh-20040203.zip">ce code</a>, un m&eacute;lange de l&#39;utilisation du SEH et des exceptions du C++ permet de prot&eacute;ger des morceaux de code dans des exceptions C++ standard. L&#39;utilisation de la fonction <font face="Courier New">_set_se_translator</font>&nbsp;permet d&#39;enregistrer une fonction appel&eacute;e lorsqu&#39;une exception survient et de transformer ces exception Win32 en exceptions C++. L&#39;exemple ici n&#39;est pas parfait puisque l&#39;original est utilis&eacute; dans le cadre de la protection d&#39;un programme complet <em>(tout le main en quelque sorte). </em>Il se pourrait qu&#39;une exception SEH lanc&eacute;e en dehors d&#39;un bloc prot&eacute;g&eacute; g&eacute;n&egrave;re tout de m&ecirc;me la boite de dialogue de plantage g&eacute;n&eacute;rique.</font> 
</p>
<p>
<font size="2">Quoi qu&#39;il en soit, l&#39;utilisation de ce code est assez simple, il faut juste noter qu&#39;il est n&eacute;cessaire d&#39;activer la g&eacute;n&eacute;ration des StackFrames<em> (d&eacute;sactiver Omit Stack Frames)</em>&nbsp;ainsi que des informations de debug sous forme de fichier ext&eacute;rieur<em> (Debug Information Format : Program Database).</em> Il faut noter &eacute;galement qu&#39;il est indispensable de distribuer le fichier <font face="Courier New">imagehlp.dll</font> et les fichiers pdb <em>(Program Database)</em>&nbsp;avec l&#39;executable pour avoir l&#39;affichage de la StackTrace lors d&#39;une exception. </font>
</p>
<p>
<font size="2">Bien entendu pour la distribution publique d&#39;un programme, on ne publiera pas les fichiers pdb, qui contiennent &eacute;norm&eacute;ment d&#39;informations.</font> 
</p>
<p>
<font size="2">Il faut noter &eacute;galement&nbsp;qu&#39;il est&nbsp;possible d&#39;utiliser le SEH sous sa forme &quot;native&quot;, en C.&nbsp;Vous aurez simplement&nbsp;&agrave; supprimer&nbsp;les fonctionnalit&eacute;s sp&eacute;cifique au C++ du code pr&eacute;c&eacute;dent.&nbsp;<em>(Je donne l&#39;info&nbsp;puisque certains utilisent encore ce langage... :p)</em></font> 
</p>
<p>
<font size="2">Bon debug !</font>
</p>

{% include imported_disclaimer.html %}
