---
layout: post
title: "ODP.NET 10.2.0.2.20 Connection Pool Race Condition"
date: 2006-12-22 12:56:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2006/12/22/ODPNET-1020220-Connection-Pool-Race-Condition-", "/post/2006/12/22/odpnet-1020220-connection-pool-race-condition-"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Verdana" size="2">Aaah, les joies d&#39;Oracle. Ma base de donn&eacute;es pr&eacute;f&eacute;r&eacute;e... accompagn&eacute;e de son cort&egrave;ge de bugs...</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Bon, j&#39;arr&ecirc;te l&agrave; le sarcasme, mais j&#39;ai du mal :)</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Dans le cadre d&#39;un projet sur lequel je travaille, je suis tomb&eacute; sur un bug tr&egrave;s, tr&egrave;s g&ecirc;nant : Le provider ODP.NET d&#39;oracle peut se connecter &agrave; un sch&eacute;ma sur lequel il n&#39;est pas cens&eacute; se connecter...</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Je m&#39;explique : dans le m&ecirc;me AppDomain d&#39;un m&ecirc;me process, je cr&eacute;e dans des Threads diff&eacute;rentes deux OracleConnection avec deux chaines de connexion diff&eacute;rentes. Jusque l&agrave; , rien d&#39;anormal. Le probl&egrave;me est que de temps &agrave; autres, l&#39;une de deux connexion va utiliser la chaine de l&#39;autre thread ! </font>
</p>
<p align="justify">
<font face="Verdana" size="2">Apr&egrave;s avoir v&eacute;rif&eacute; rapidement le contenu de l&#39;objet OracleConnection, il se trouve qu&#39;une variable nomm&eacute;e &quot;m_InternalConStr&quot; contient parfois une chaine qui ne correspond par du tout &agrave; celle qui a &eacute;t&eacute; pass&eacute;e en param&egrave;tre dans le constructeur... C&#39;est tr&egrave;s g&eacute;nant, car si l&#39;on se connecte &agrave; une base alors que l&#39;on pense se connecter &agrave; une autre, ... je vous laisse immaginer les d&eacute;gats si on fait des updates.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Donc apr&egrave;s de nombreuses tentatives, je suis arriv&eacute; &agrave; la conclusion qu&#39;il faut synchroniser tous&nbsp;les appels &agrave; OracleConnection du constructeur jusqu&#39;&agrave;&nbsp;Open avec un Mutex, et cela sur l&#39;AppDomain courant. Vous imaginez ais&eacute;ment&nbsp;le goulot d&#39;&eacute;tranglement. J&#39;aurais tout aussi bien pu d&eacute;sactiver le Connection Pool, mais la aussi, cot&eacute; performance cela devient g&ecirc;nant.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Bien entendu, ce genre de probl&egrave;me apparait plus souvent sur une machine Multi Processeur. (En production dans mon cas, ca fait toujours plaisir...)</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Je n&#39;arrive pas&nbsp;&agrave; reproduire la Race Condition de mani&egrave;re syst&eacute;matique, mais je met avec ce post un exemple de code qui teste tout ca. G&eacute;n&eacute;ralement, l&#39;erreur apparait au bout de quelques essais. Pour tester si le ConnectionPool est consistant, j&#39;effectue quelques lignes de Reflection pour aller chercher des variables interne... pas tr&egrave;s propre, mais c&#39;est suffisement d&eacute;terministe.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Si quelqu&#39;un se sent suffisement motiv&eacute; pour tester... :)</font>
</p>

{% include imported_disclaimer.html %}
