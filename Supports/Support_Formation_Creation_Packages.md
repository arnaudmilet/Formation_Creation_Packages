---
title: "Création de packages sur RStudio"
author: Arnaud MILET
date: 2019
output: 
  html_document:
    toc: true
    toc_float: true
    number_sections: true
    keep_md: true
  
---



******

#	Utilité des packages sur R

**Etendre les fonctionnalités de R et simplifier l’utilisation du code**

Un package R est un mécanisme permettant d'étendre les fonctionnalités de base de R. 

L'utilisation de fonctions simplifie les choses pour l'utilisateur, qui n'a plus besoin de connaître les détails du code. Ils doivent seulement connaître les arguments à entrer et les résultats en sortie.

**Regrouper les familles de fonctions**

Une fois que plusieurs fonctions ont été développées, il devient naturel de les regrouper dans des collections de fonctions visant à atteindre un objectif global. Cet ensemble de fonctions peut être assemblé dans un package R pour former une famille de fonctions. 

**Partager**

Les packages R constituent le meilleur moyen de distribuer du code à d’autres utilisateurs. Les packages R sont écrits dans un format normalisé. Les utilisateurs de R sont déjà familiarisés avec l’utilisation de l'installation ou de l'aide sur des fonctions et pourront donc rapidement adopter votre code s’il est présenté dans ce format.

**Tester le code**

Les différents outils fournis avec R (et RStudio) permettent de contrôler vos packages afin d’éviter toute incohérence ou erreur. 

Des détails sur la construction de packages vous seront apportés dans le manuel Writing R Extensions[^1] disponible sur le [CRAN](https://cran.r-project.org), dans le livre R Packages[^2] d'Hadley Wickham ou encore dans le livre Mastering Software Development in R de Roger D. Peng, Sean Kross et Brooke Anderson[^3] disponibles en ligne. L'ensemble des notes ici s'inspire très largement de ces 3 sources (souvent une simple traduction) que complètent quelques sources très intéressantes sur le partage de packages via Github. 

#	Un exemple de package
##	Cartography de Timothé Giraud

Un exemple que j'aime bien :  
<"https://github.com/riatelab/cartography">  
parce que :  
1.  Je sais ce qu'il fait  
2.  Il est bien structuré  

##	Compréhension de la structures de base d’un package

Un package R commence sa vie comme un répertoire sur votre ordinateur. Ce répertoire a une disposition spécifique avec des fichiers et des sous-répertoires spécifiques. Les deux sous-répertoires requis sont:  

* **R**, qui contient tous vos fichiers de code R  
* **man**, qui contient vos fichiers de documentation.

Au niveau supérieur de votre répertoire de package, vous aurez un fichier DESCRIPTION et un fichier NAMESPACE. Cela représente la configuration minimale requise pour un package R. D'autres fichiers et sous-répertoires peuvent néanmoins être ajoutés.

Si RStudio n'est pas directement conçu pour créer des packages R, 2 packages permettent d'étendre ses fonctionnalités et facilitent assez largement la création de nouveaux packages:

* **devtools**  
* **roxygen2** 

###	Descriptions

Le fichier DESCRIPTION est une partie essentielle d'un package R car il contient des métadonnées clés pour le package utilisées par les repertoires tels que CRAN et par R. En particulier, ce fichier contient le **nom du package**, **le numéro de version**, les informations de contact de **l'auteur**, les informations de **la licence**, ainsi que toutes **les dépendances à d'autres packages**.

À titre d’exemple, voici le fichier DESCRIPTION du packageage mvtsplot sur CRAN. Ce package fournit une fonction permettant de tracer des données de séries temporelles multivariées.

```
Package:  mvtsplot
Version:  1.0-3
Date:  2016-05-13
Depends:  R (>= 3.0.0)
Imports: splines, graphics, grDevices, stats, RColorBrewer
Title:  Multivariate Time Series Plot
Author:  Roger D. Peng <rpeng@jhsph.edu>
Maintainer:  Roger D. Peng <rpeng@jhsph.edu>
Description:  A function for plotting multivariate time series data.
License:  GPL (>= 2)
URL: https://github.com/rdpeng/mvtsplot
```
###	NAMESPACE

Le fichier NAMESPACE spécifie l'interface du package présenté à l'utilisateur. Cela se fait via une série d'instructions export(), qui indiquent quelles fonctions du package sont exportées vers l'utilisateur. Les fonctions non exportées ne peuvent pas être appelées directement par l'utilisateur. Outre les exportations, le fichier NAMESPACE spécifie également les fonctions ou packages importés par le package. Si votre package dépend de fonctions d'un autre package, vous devez les importer via le fichier NAMESPACE.

Vous trouverez ci-dessous le fichier NAMESPACE du packageage mvtsplot décrit ci-dessus.

```
export("mvtsplot")

import(splines)
import(RColorBrewer)
importFrom("grDevices", "colorRampPalette", "gray")
importFrom("graphics", "abline", "axis", "box", "image", "layout",
           "lines", "par", "plot", "points", "segments", "strwidth",
           "text", "Axis")
importFrom("stats", "complete.cases", "lm", "na.exclude", "predict",
           "quantile")

```
Nous pouvons voir ici qu’une seule fonction est exportée du package (la fonction mvtsplot()). Il existe deux types d'instructions d'importation:  

*  import(), prend simplement un nom de package comme argument, et l'interprétation est que toutes les fonctions exportées depuis ce package externe seront accessibles à votre package
*  importFrom(), prend comme argument un package et une série de noms de fonctions. Cette directive vous permet de spécifier exactement la fonction dont vous avez besoin depuis un package externe. Par exemple, ce package importe les fonctions colorRampPalette() et gray() du package grDevices.

De manière générale, il est préférable d’utiliser importFrom() et de spécifier la fonction dont vous avez besoin à partir d’un package externe. Cependant, dans certains cas, lorsque vous avez vraiment besoin de presque toutes les fonctions d’un package, il peut être plus efficace de simplement importer() l’ensemble du package.

En ce qui concerne l'exportation de fonctions, il est important de bien réfléchir aux fonctions que vous souhaitez exporter. Avant tout, les fonctions exportées doivent être documentées et prises en charge. Les utilisateurs s'attendent généralement à ce que les fonctions exportées soient présentes dans les itérations suivantes du package. Il est généralement préférable de limiter le nombre de fonctions que vous exportez (si possible). Il est toujours possible d’exporter ultérieurement un élément si cela est nécessaire, mais le fait de supprimer une fonction exportée une fois que les utilisateurs sont habitués à l’avoir disponible peut perturber les utilisateurs. Enfin, l’exportation d’une longue liste de fonctions a pour effet d’encombrer l’espace de noms d’un utilisateur avec des noms de fonctions pouvant entrer en conflit avec des fonctions d’autres packages. Réduire au minimum le nombre d'exportations réduit les risques de conflit avec d'autres packages (l'utilisation de noms de fonction plus spécifiques à un package est un autre moyen).


####	Appel des fonctions externes au package

Lorsque vous commencez à utiliser plusieurs packages dans R, la probabilité que deux fonctions aient le même nom augmente. Par exemple, le package dplyr couramment utilisé a une fonction nommée filter(), qui est également le nom d’une fonction du package stats. Si on a les deux packages chargés (un scénario plus que probable), comment sait-on exactement quelle fonction filter() ils faut appeler?

Dans R, chaque fonction a un nom complet, qui inclut le nom du package dans le nom:

```
<package name>::<exported function name>

```

Par exemple, la fonction filter() du package dplyr peut être appelée sous la forme dplyr::filter(). De cette façon, il n’ya aucune confusion sur la fonction filter() que nous appelons. Bien qu'en principe, chaque fonction puisse être appelée de cette manière, cela peut être fastidieux pour un travail interactif. Cependant, pour la programmation, il est souvent plus sûr d'appeler une fonction en utilisant le nom complet.

###	R

Le sous-répertoire R contient tout votre code R, soit dans un seul fichier, soit dans plusieurs fichiers. Pour les packages plus volumineux, il est généralement préférable de scinder le code en plusieurs fichiers regroupant logiquement les fonctions. Les noms des fichiers de code R n’importe pas, mais il n’est généralement pas judicieux d’avoir des espaces dans les noms de fichiers.

###	man

Le sous-répertoire man contient les fichiers de documentation de tous les objets exportés d'un package. Avec les anciennes versions de R, il fallait écrire la documentation des objets R directement dans le répertoire man en utilisant une notation de type LaTeX. Cependant, avec le développement du package roxygen2, nous n’avons plus besoin de le faire et nous pouvons écrire la documentation directement dans les fichiers de code R. Par conséquent, vous aurez probablement peu d'interaction avec le répertoire man, car tous les fichiers qu'il contient seront automatiquement générés par le packageage roxygen2. 

##	Votre premier package en quelques minutes[^4]

**Étape 1 : Création du projet Rstudio**

Grâce à Rstudio et des packages tels que {devtools}, {usethis} et {roxygen2}, concevoir un package revient à cliquer sur File > New Project > New Directory > R package using devtools.

Assurez-vous d’avoir bien installé les packages suivants :


```r
install.packages(c("devtools", "usethis", "roxygen2"))
```

![](Images/Crea_projet.png)
![](Images/Crea_projet2.png)



```r
available::available("monpackage")
```

![](Images/Crea_projet3.png)
![](Images/Crea_projet4.png)

RStudio s’ouvre alors sur un nouveau projet avec un nouveau dossier contenant une arborescence dont il suffit de retenir deux choses :

*  le fichier DESCRIPTION qu’il faut remplir à la main en remplacant les valeurs pré-remplies (nom, licence, etc)
*  le dossier R dans lequel il faudra placer ses fonctions

**Étape 2 : le fichier DESCRIPTION**

Ouvrir le fichier et l’éditer.
Vous pouvez voir ce fichier comme un texte à trou dans lequel il faut entrer votre nom, prénom, décrire ce que fait le package, etc. Vous devez choisir une licence (GPL-3, MIT).

Une fois ce fichier complété, cliquez sur check dans l’onglet build en haut à droite dans Rstudio.

![](Images/check.png)

Une cinquantaine de tests seront alors réalisés. Dans la mesure où votre package est vide, il est essentiel d’avoir 0 erreur, 0 warning et 0 note.

**Étape 3 : Création de fonctions**

Vous pouvez maintenant commencer à ranger vos fonctions dans le dossier R de l’arborescence, dans des fichiers portant l’extension .R.


```r
moyenne <- function(x){
  x <- x %>% na.omit()
  sum(x)/length(x)
}
```

**Étape 4 : Documentation et NAMESPACE**

En cliquant sur check à l’étape 2, vous avez fabriqué un dossier man dans l’arborescence. Mais pour l’instant il est vide…

![](Images/man.png)

Le fichier NAMESPACE comme le dossier man ne seront pas le centre de notre attention. Le NAMESPACE sert à définir comment notre package interagit avec le monde extérieur (importer d’autres fonctions, d’autres packages, exporter telle ou telle fonction) et le dossier man contient la documentation des fonctions. En pratique ils seront générés et mis à jour automatiquement grâce à {roxygen2}, en cliquant sur le bouton check.

Afin que le NAMESPACE et man soient gérés automatiquement, nous allons adopter un système de balises spécifiques pour documenter le code R. Des commentaires un peu particuliers vont commencer, non pas par #, mais par #'


```r
#' moyenne d’un vecteur
#' Une fonction pour faire une moyenne en enlevant les valeurs manquantes
#'
#' @param x un vecteur numerique
#'
#' @return la fonction renvoie la moyenne d'un vecteur
#' @import magrittr
#' @importFrom stats na.omit
#' moyenne(c(4,5))
#' @export
moyenne <- function(x){
  x <- x %>% na.omit()
  sum(x)/length(x)
}
```

Vous pouvez génèrer directement un template de documentation. On peut ainsi aller dans Code > Insert Roxygen Skeleton :

![](Images/skeleton_roxygen.png)

La fonction moyenne ici en exemple utilise le %>%, il faut donc lier notre package au package {magrittr} qui contient cet opérateur. De même la façon na.omit vient du package {stats}. Notre package dépend donc de {magrittr} et de {stats}. Un utilisateur qui ne disposerait pas de {magrittr} ou {stats} ne pourrait pas utiliser notre fonction !

C’est @importFrom qui permet d’aller chercher dans le package {stats} la fonction na.omit utilisée, et @import va importer l’intégralité des fonctions de {magrittr}.

@import importe TOUT le package tandis que @importFrom permet de n’importer qu’un ensemble de fonctions. C’est @importFrom qui sera privilégié car c’est le niveau le plus fin.

MAIS cela ne suffit pas. Il faut aussi modifier le fichier DESCRIPTION pour y ajouter la dépendance aux deux packages {magrittr} et {stats} dans la rubrique Imports. En l’état, un check renverra des warnings/erreurs. (vous pouvez tester !)

Cependant, modifier le fichier DESCRIPTION à la main est source d'erreurs. Un simple espace au mauvais endroit peut générer des erreurs. Nous allons donc privilégier l’usage de {devtools}.


```r
devtools::use_package("stats")
devtools::use_package("magrittr")
```

Le fichier DESCRIPTION ressemble maintenant à ceci :

```
Package: monpackage
Title: une demonstration
Version: 0.0.0.9000
Authors@R: person("Arnaud", "Milet", email = "arnaud.milet@f-sidd.com", role = c("aut", "cre"))
Description: il s agt d un test de demonstration.
Depends: R (>= 3.5.1)
License: GPL-3
Encoding: UTF-8
LazyData: true
RoxygenNote: 6.1.0
Imports: stats,
    magrittr

```


Le package est créé! Faites un check puis cliquez dans l’onglet build sur Install and Restart. R redémarre et lance votre package (on notera le library(monpackage) lancé pour nous dans la console). La fonction moyenne est utilisable et taper ?moyenne revient à interroger R sur l’aide de la fonction : une page de documentation du plus bel effet est ouverte dans l’onglet help.

**Étape 5 : Partager son travail**
Dans l’onglet Build se trouve un bouton more qui vous permet de construire votre package source (Build Source Package).

![](Images/build.png)

Cela génère un fichier compressé en tar.gz (ici monpackage_0.0.0.9000.tar.gz) que vous pouvez partager avec qui veut… voire l’envoyer au CRAN pour le rendre disponible à la communauté.


##	Exercice: Créer un package affichant votre nom en console


#	Package devtools 

Le développement de packages R est devenu considérablement plus facile ces dernières années avec l'introduction d'un package par Hadley Wickham appelé devtools. Comme le nom du package l'indique, cela inclut diverses fonctions facilitant le développement de logiciels dans R.

Voici quelques-unes des fonctions clés incluses dans devtools et leur fonction, dans l'ordre dans lequel vous les utiliserez probablement lorsque vous développez un package R:


Fonction            Usage                                                                                                                                                                                                                                                                                                                                 
------------------  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
load_all            Charge le code de toutes les fonctions du package.                                                                                                                                                                                                                                                                                    
document            Créer d'une part le répertoire \man et ses fichiers de documentation sur les fonctions et d'autre part le fichier «NAMESPACE» à partir des codes roxygen2 repérés                                                                                                                                                                     
use_data            Enregistrer un objet contenu dans votre session R en tant que données appartenant au package                                                                                                                                                                                                                                          
use_vignette        Configure le package pour inclure une vignette                                                                                                                                                                                                                                                                                        
use_readme_rmd      Configure le package pour inclure un fichier README au format Rmarkdown                                                                                                                                                                                                                                                               
use_build_ignore    Spécifiez ici les fichiers à ignorer lors de la création du package R (par exemple, si vous avez un dossier dans lequel vous rédigez un article de journal sur le package, vous pouvez inclure tous les fichiers associés dans un dossier que vous avez défini comme étant ignoré pendant le processus de la construction du package) 
check               Vérifiez dans le package R s'il existe d'éventuels avertissements : ERROR, WARNING ou NOTE                                                                                                                                                                                                                                            
build_win           Construisez une version du package pour Windows et envoyez-la à vérifier sur un ordinateur Windows. Vous recevrez un email avec un lien vers les résultats.                                                                                                                                                                           
use_travis          Configure le package pour faciliter l'utilisation de Travis CI avec le package                                                                                                                                                                                                                                                        
use_cran_comments   Créez un fichier dans lequel vous pouvez ajouter des commentaires à inclure lors de votre soumission CRAN.                                                                                                                                                                                                                            
submit_cran         Soumettre le package à CRAN                                                                                                                                                                                                                                                                                                           
use_news_md         Ajouter un fichier au package pour indiquer les changements dans les nouvelles versions                                                                                                                                                                                                                                               

 
Certaines de ces fonctions ne devront être utilisées qu’une fois pour un package. Les fonctions uniques (par package) sont principalement celles qui configurent un certain type d’infrastructure pour le package. Par exemple, si vous souhaitez utiliser R Markdown pour créer un fichier README pour un package que vous publiez sur GitHub, vous pouvez créer l'infrastructure appropriée à l'aide de la fonction use_readme_rmd. Cette fonction ajoute un fichier  README dans le répertoire principal du package avec le nom «README.Rmd». Vous pouvez éditer ce fichier et le rendre à Markdown pour fournir aux utilisateurs de GitHub plus d'informations sur votre package. Toutefois, vos vérifications CRAN poseront des problèmes s’il existe un fichier README dans ce répertoire de niveau supérieur du package. Par conséquent, la fonction use_readme_rmd ajoute également les noms de fichier du fichier README R Markdown, ainsi que le fichier Markdown créé, dans le fichier “.Rbuildignore”, donc il n’est pas inclus lors de la construction du package.


#	Documentation

Il existe deux principaux types de documentation que vous pouvez inclure avec les packages:

*  Des documents plus longs qui donnent des tutoriels ou des aperçus pour l'ensemble du package  
*  Fichiers d'aide plus courts et spécifiques à chaque fonction pour chaque fonction ou groupe de fonctions associées

Vous pouvez créer le premier type de document à l'aide de vignettes de package, de fichiers README ou des deux. Pour les fichiers d’aide spécifiques à une fonction, le moyen le plus simple de les créer consiste à utiliser le package roxygen2.

Dans cette section, nous expliquerons pourquoi et comment créer cette documentation. En outre, la documentation de vignette / README peut être réalisée à l'aide de knitr pour créer des documents R Markdown qui mélangent du code R et du texte. Nous allons donc inclure plus de détails sur ce processus.

##	Vignette et README

Vous voudrez probablement créer un document qui explique aux utilisateurs comment utiliser votre package. Vous pouvez le faire à travers deux formats:

*   **Vignette**: Ce document est fourni avec votre package R, il devient donc localement disponible pour un utilisateur une fois qu'il installe votre package depuis CRAN. Ils l'auront également disponible s'ils installent le package à partir de GitHub, à condition qu'ils utilisent build_vignettes = TRUE lors de l'exécution de install_github.

*  **Fichier README**: Si votre package est sur GitHub, ce document sera affiché sur la page principale du repertoire.

Un package n'a besoin d'un fichier README que si vous postez le package sur GitHub. Pour tout repertoire GitHub, s'il existe un fichier README.md dans le répertoire supérieur du repertoire, il sera rendu sur la page principale du repertoire GitHub sous le contenu du repertoire répertorié. Pour un exemple, visitez https://github.com/leighseverson/countyweather et faites défiler vers le bas. Vous verrez une liste de tous les fichiers et sous-répertoires inclus dans le repertoire du package, ainsi que le contenu du fichier README.md du package, qui fournit un didacticiel sur l’utilisation du package.

Si le fichier README n'a pas besoin d'inclure le code R, vous pouvez l'écrire directement sous forme de fichier .md, à l'aide de la syntaxe Markdown. Si vous souhaitez inclure du code R, vous devez commencer par un fichier README.Rmd, que vous pouvez convertir en Markdown à l'aide de knitr. Vous pouvez utiliser le package devtools pour ajouter un fichier README.md ou README.Rmd à un répertoire de package, en utilisant respectivement use_readme_md ou use_readme_rmd. Ces fonctions ajouteront le fichier approprié au niveau supérieur du répertoire du package et ajouteront également le nom du fichier à «.Rbuildignore», car la présence de l’un de ces fichiers au niveau supérieur du répertoire du package risquerait sinon de poser problème lors de la création du package.

Le fichier README est un moyen utile de fournir aux utilisateurs de GitHub des informations sur votre package, mais il ne sera pas inclus dans les versions du package ou ne sera pas disponible via CRAN pour les packages qui y sont postés. Si vous souhaitez créer des didacticiels ou des documents de synthèse inclus dans une construction de package, vous devez le faire en ajoutant une ou plusieurs vignettes de package. Les vignettes sont stockées dans un sous-répertoire de vignettes dans le répertoire du package.

Pour ajouter un fichier de vignette, enregistré dans ce sous-répertoire (qui sera créé si vous ne l'avez pas déjà), utilisez la fonction use_vignette de devtools. Cette fonction prend comme argument le nom de fichier de la vignette que vous souhaitez créer et le package pour lequel vous souhaitez le créer (la valeur par défaut est le package dans le répertoire de travail en cours). Par exemple, si vous travaillez actuellement dans le répertoire de niveau supérieur de votre package et que vous souhaitez ajouter une vignette appelée "model_details", vous pouvez le faire avec le code:


```r
use_vignette("model_details")
```

Vous pouvez avoir plus d'une vignette par package, ce qui peut être utile si vous souhaitez inclure une vignette donnant un aperçu plus général du package, ainsi que quelques vignettes plus détaillées sur des aspects ou des applications particuliers.

Une fois que vous avez créé une vignette avec use_vignette, veillez à mettre à jour l’entrée d’indexation de la vignette dans le YAML de la vignette (le code situé en haut du document R Markdown). Remplacez «Titre de la vignette» par le titre que vous utilisez pour la vignette.

##	Rmarkdown et Knitr

Les fichiers vignette et README peuvent être écrits en tant que fichiers RMarkdown, ce qui vous permettra d’inclure des exemples de code R et des résultats de votre package. L'un des outils les plus intéressants de R est le système knitr qui combine code et texte pour créer un document reproductible.

Les fichiers RMarkdown sont principalement écrits à l'aide de Markdown. Pour écrire des fichiers RMarkdown, vous devez comprendre ce que sont les langages de marquage comme Markdown et leur fonctionnement. Ou pas... Dans Word et les autres programmes de traitement de texte que vous avez utilisés, vous pouvez ajouter une mise en forme à l’aide de boutons et de raccourcis clavier (par exemple, «Ctrl-B» pour indiquer en gras). Le fichier enregistre les mots que vous tapez. Il enregistre également la mise en forme, mais vous voyez la sortie finale, plutôt que le balisage de mise en forme, lorsque vous modifiez le fichier (WYSIWYG - What You See is What You Get). En revanche, dans les langages de marquage, vous marquez directement le document pour indiquer le formatage de la version finale (par exemple, vous tapez `**` **bold** `**` dans le fichier pour obtenir un document en gras). Voici des exemples de langages de balisage:

*  HTML
*  Latex
*  Markdown

Pour écrire un fichier en Markdown, vous devez connaître les conventions de création du formatage. Vous pouvez consulter le guide du RMarkdown[^5] ou plus simplement la cheatsheet RMarkdown[^6].


Le début d'un fichier Markdown fournit des métadonnées pour le fichier (auteurs, titre, format) dans un langage appelé YAML. Par exemple, la section YAML d'une vignette de package pourrait ressembler à ceci:  

```
---
title: "Model Details for example_package"
author: "Jane Doe"
date: "2016-11-08"
output: rmarkdown::html_vignette
vignette: >
  %\VignetteIndexEntry{Model Details for example_package}
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteEncoding{UTF-8}
---
```

Lors de la création de documents R Markdown à l'aide de la barre d'outils RStudio, une grande partie de ce YAML sera automatiquement générée en fonction de vos spécifications lors de l'ouverture du fichier initial. Cependant, ce n’est pas le cas des vignettes de packages, pour lesquelles vous devrez aller dans le YAML et ajouter les auteurs et le titre vous-même. Conservez par défaut le moteur de vignette, l'encodage, la sortie et la date de la vignette.

Pour plus de conventions de Markdown, consultez le Guide de référence de RStudio R Markdown (lien également disponible dans «Aide» de RStudio).

Les fichiers R Markdown fonctionnent  comme les fichiers Markdown, mais ajoutent la possibilité d'inclure du code R qui sera exécuté avant le rendu du document final. La sortie finale aura les résultats de votre code et du texte normal.

Les étapes de base pour ouvrir et restituer un fichier R Markdown dans RStudio sont les suivantes:  

1.  Pour ouvrir un nouveau fichier R Markdown, allez dans "File" -> "New file" -> "RMarkdown ..." -> pour l'instant, choisissez un "Document" au format "HTML".  
2. Cela ouvrira un nouveau fichier R Markdown dans RStudio. L'extension de fichier pour les fichiers R Markdown est «.Rmd».  
3.  Le nouveau fichier est livré avec un exemple de code et de texte. Vous pouvez exécuter le fichier tel quel pour essayer l'exemple. Vous allez finalement supprimer cet exemple de code et de texte et le remplacer par le vôtre.  
4.  Une fois que vous avez «tricoté» le fichier R Markdown, R rendra un fichier HTML avec la sortie. Ceci est automatiquement enregistré dans le même répertoire que celui dans lequel vous avez enregistré votre fichier .Rmd.  
5.  Ecrivez tout sauf le code R en utilisant la syntaxe Markdown.

La fonction knit du package knitr fonctionne en prenant un document au format R Markdown (parmi quelques formats possibles), en le lisant pour tous les marqueurs du début du code R, en exécutant n’importe quel code entre ce marqueur «début» et un marqueur. marqueur indiquant un retour à Markdown normal, écrivant les résultats pertinents du code R dans le fichier Markdown au format Markdown, puis transmettant l'intégralité du document, maintenant au format Markdown au lieu du format R Markdown, à un logiciel capable de restituer Markdown au format de sortie souhaité (par exemple, compiler un document PDF, Word ou HTML).

Cela signifie que tout ce qu'un utilisateur doit faire pour inclure du code R, qui sera exécuté si nécessaire, dans un document consiste à le séparer correctement des autres parties du document au moyen des marqueurs appropriés. Pour indiquer le code R dans un document RMarkdown, vous devez séparer le bloc de code à l'aide de la syntaxe suivante:

` ```{r} `  
`my_vec <- 1:10`    
` ```  `  

Cette syntaxe indique à R comment trouver le début et la fin des éléments de code R lorsque le fichier est rendu. R passera en revue, trouvera chaque morceau de code R, l'exécutera et créera une sortie (sortie imprimée ou chiffres, par exemple), puis passera le fichier à un autre programme pour terminer le rendu (par exemple, Tex pour les fichiers pdf).

Vous pouvez spécifier un nom pour chaque morceau, si vous le souhaitez, en l’insérant après «r» lorsque vous commencez votre morceau. Par exemple, pour attribuer le nom load_mtcars à un fragment de code qui charge le jeu de données mtcars, spécifiez ce nom au début du fragment de code:

` ```{r load_mtcars} `  
` data(mtcars)`  
` ```  `  

Voici quelques conseils pour nommer des fragments de code:  

*  Les noms de morceaux doivent être uniques dans un document.  
*  Tous les morceaux que vous ne nommez pas sont numérotés par knitr.  

Vous n'êtes pas obligé de nommer chaque morceau. Cependant, il y a quelques avantages:  

*  Il sera plus facile de trouver des erreurs.  
*  Vous pouvez utiliser les étiquettes de bloc pour référencer des étiquettes de figure.  
*  Vous pouvez référencer des morceaux plus tard par nom.  

### Quelques options knitr

Vous pouvez également ajouter des options lorsque vous démarrez un morceau. Beaucoup de ces options peuvent être définies comme VRAI / FAUX.

Pour inclure l'une de ces options, ajoutez l'option et la valeur dans les parenthèses d'ouverture, puis séparez les multiples options par des virgules:

`  ```{r  messages = FALSE, echo = FALSE}`   
`  mtcars[1, 1:3]`   
` ```  `  

Vous pouvez définir des options «globales» au début du document. Cela créera de nouvelles valeurs par défaut pour tous les morceaux du document. Par exemple, si vous souhaitez que écho, avertissement et message soient FAUX par défaut dans tous les fragments de code, vous pouvez exécuter:

` ```{r  global_options}`   
` knitr::opts_chunk$set(echo = FALSE, message = FALSE, warning = FALSE)`   
` ```  `  

Si vous définissez à la fois les options de bloc globales et locales que vous avez définies spécifiquement pour un bloc, celles-ci auront priorité. Par exemple, exécuter un document avec:

`  ```{r  global_options}`   
`  knitr::opts_chunk$set(echo = FALSE, message = FALSE, warning = FALSE)`  
` ```  `  

` ```{r  check_mtcars, echo = TRUE}`   
` head(mtcars, 1)`   
` ```  `  

afficherait le code du bloc check_mtcars, car l’option spécifiée pour ce bloc spécifique (echo = TRUE) remplacerait l’option globale (echo = FALSE).

Vous pouvez également inclure la sortie R directement dans votre texte à l’aide de: ``

Il y a des observations ` ` r nrow (mtcars) ` ` dans le jeu de données mtcars. La moyenne des miles par gallon est «moyenne» (mtcars $ mpg, na.rm = TRUE).

Une fois le fichier rendu, cela donne:

Il y a 32 observations dans le jeu de données mtcars. La moyenne de miles par gallon est de 20,090625.

Voici quelques conseils qui vous aideront à diagnostiquer certains problèmes lors du rendu des fichiers R Markdown:

*  Veillez à enregistrer votre fichier R Markdown avant de l'exécuter.  
*  Tout le code du fichier fonctionnera «à partir de zéro», comme si vous veniez d'ouvrir une nouvelle session R.  
*  Le code sera exécuté en utilisant comme répertoire de travail le répertoire dans lequel vous avez enregistré le fichier R. Markdown.  

Vous voudrez essayer des morceaux de votre code pendant que vous écrivez un document R Markdown. Vous pouvez le faire de plusieurs manières:

*  Vous pouvez exécuter du code en morceaux, tout comme vous pouvez exécuter du code à partir d'un script (Ctrl-Retour ou le bouton «Exécuter»).  
*  Vous pouvez exécuter tout le code dans un bloc (ou tout le code dans tous les morceaux) en utilisant les différentes options du bouton "Exécuter" de RStudio.  
*  Toutes les options "Exécuter" ont des raccourcis clavier, vous pouvez donc les utiliser.  

Vous pouvez utiliser ce format pour créer de la documentation, y compris des vignettes, afin de donner aux utilisateurs des conseils et des exemples d'utilisation de votre package.


##	roxygen2 et les fichiers d'aide

Outre la rédaction de didacticiels donnant une vue d'ensemble de l'ensemble de votre package, vous devez également rédiger une documentation spécifique indiquant aux utilisateurs comment utiliser et interpréter les fonctions que les utilisateurs devraient appeler directement.

Ces fichiers d’aide iront finalement dans un dossier appelé / man de votre package, dans un format de documentation R (extensions de fichier .Rd) assez similaire à LaTex. Vous deviez écrire tous ces fichiers séparément. Cependant, le package roxygen2 vous permet de placer toutes les informations d'aide directement dans le code où vous définissez chaque fonction.

Avec roxygen2, vous ajoutez les informations du fichier d’aide directement au-dessus du code où vous définissez chaque fonction, dans les scripts R enregistrés dans le sous-répertoire R du répertoire du package. Vous commencez chaque ligne de la documentation roxygen2 par # '(le deuxième caractère est une apostrophe, pas un backtick). La première ligne de la documentation doit donner un titre court pour la fonction, et le prochain bloc de documentation devrait être une description plus longue. Après cela, vous utiliserez des balises commençant par @ pour définir chaque élément que vous incluez. Vous devez laisser une ligne vide entre chaque section de la documentation et vous pouvez utiliser l'indentation pour les lignes d'éléments suivantes et ultérieures afin de faciliter la lecture du code.

Voici un exemple de base de la manière dont cette documentation roxygen2 rechercherait une fonction simple «Hello world»:

```  
#' Print "Hello world" 
#'
#' This is a simple function that, by default, prints "Hello world". You can 
#' customize the text to print (using the \code{to_print} argument) and add
#' an exclamation point (\code{excited = TRUE}).
#'
#' @param to_print A character string giving the text the function will print
#' @param excited Logical value specifying whether to include an exclamation
#'    point after the text
#' 
#' @return This function returns a phrase to print, with or without an 
#'    exclamation point added. As a side effect, this function also prints out
#'    the phrase. 
#'
#' @examples
#' hello_world()
#' hello_world(excited = TRUE)
#' hello_world(to_print = "Hi world")
#'
#' @export
hello_world <- function(to_print = "Hello world", excited = FALSE){
    if(excited) to_print <- paste0(to_print, "!")
    print(to_print)
}
```  

##	Quelques tags communs de roxygen2


```r
tag_roxygen2<-data.frame(
  tag=c("@return",
        "@parameter",
        "@inheritParams",
        "@examples",
        "@details",
        "@note",
        "@source",
        "@references",
        "@importFrom",
        "@export"
        
        ),
  usage=c("Une description de l'objet retourné par la fonction",
          "Explication d'un paramètre de fonction",
          "Nom d'une fonction à partir de laquelle obtenir les définitions de paramètres",
          "Exemple de code montrant comment utiliser la fonction",
          "Ajoute plus de détails sur le fonctionnement de la fonction (par exemple, les spécificités de l'algorithme utilisé)",
          "Ajoute des notes sur la fonction ou son utilisation",
          "Ajouter des détails sur la source du code ou des idées pour la fonction",
          "Ajoutez toutes les références pertinentes à la fonction.",
          "Importe une fonction d'un autre package à utiliser dans cette fonction (cela est particulièrement utile pour les fonctions en ligne telles que%>% et% within%)",
          "Exporte la fonction pour que les utilisateurs y aient un accès direct lors du chargement du package.")
)

knitr::kable(tag_roxygen2)
```



tag              usage                                                                                                                                                          
---------------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------
@return          Une description de l'objet retourné par la fonction                                                                                                            
@parameter       Explication d'un paramètre de fonction                                                                                                                         
@inheritParams   Nom d'une fonction à partir de laquelle obtenir les définitions de paramètres                                                                                  
@examples        Exemple de code montrant comment utiliser la fonction                                                                                                          
@details         Ajoute plus de détails sur le fonctionnement de la fonction (par exemple, les spécificités de l'algorithme utilisé)                                            
@note            Ajoute des notes sur la fonction ou son utilisation                                                                                                            
@source          Ajouter des détails sur la source du code ou des idées pour la fonction                                                                                        
@references      Ajoutez toutes les références pertinentes à la fonction.                                                                                                       
@importFrom      Importe une fonction d'un autre package à utiliser dans cette fonction (cela est particulièrement utile pour les fonctions en ligne telles que%>% et% within%) 
@export          Exporte la fonction pour que les utilisateurs y aient un accès direct lors du chargement du package.                                                           

Voici quelques éléments à garder à l’esprit lorsque vous écrivez des fichiers d’aide avec roxygen2:

Les balises @example et @examples font des choses différentes. Vous devez toujours utiliser la balise @examples (pluriel) comme exemple de code, sinon vous obtiendrez des erreurs lors de la compilation de la documentation.

La fonction @inheritParams peut vous faire gagner beaucoup de temps, car si vous utilisez les mêmes paramètres dans plusieurs fonctions de votre package, vous pouvez écrire et éditer ces descriptions de paramètres au même endroit. Cependant, gardez à l'esprit que vous devez pointer @inheritParams vers la fonction où vous définissiez les paramètres à l'origine à l'aide de @param, pas une autre fonction pour laquelle vous utilisez les paramètres mais que vous définissez à l'aide d'un pointeur @inheritParams.

Si vous souhaitez que les utilisateurs puissent utiliser directement cette fonction, vous devez inclure @export dans votre documentation roxygen2. Si vous avez écrit une fonction mais constatez qu’elle n’est pas trouvée lorsque vous essayez de compiler un fichier README ou une vignette, un coupable courant est que vous avez oublié d’exporter la fonction.

Avec les balises roxygen2, vous pouvez inclure des liens, equations, listes à puces,... Voici quelques-unes des balises de formatage courantes que vous pouvez utiliser: 


```r
tag_roxygen2<-data.frame("tag format"=c("\ code{}",
                                        "\ dontrun{}",
                                        "\ link{}",
                                        "\ eqn{}",
                                        "\ deqn{}",
                                        "\ itemize{}",
                                        "\ url{}",
                                        "\ href{}"),
                         Usage=c("Mettre en forme une police de caractères pour ressembler à du code",
                  "A utiliser avec des exemples pour éviter d'exécuter le code d'exemple lors de la construction et du test d'un package",
                  "Lien vers une autre fonction R",
                  "Inclure une équation en ligne",
                  "Inclure une équation à afficher (c'est-à-dire affichée sur sa propre ligne)",
                  "Crée une liste détaillée",
                  "Inclure un lien Web",
                  "Inclure un lien Web avec un affichage différent"
                  )
)
```


Quelques exemples issus de la vignette du package roxygen2[^7]:  

*  Mise en forme des chaînes de caractères:

```
    \emph{italics}
    \strong{bold}
    \code{r_function_call(with = "arguments")}, \code{NULL}, \code{TRUE}
    \pkg{package_name}

```
*  Liens:   
    +  Vers une autre documentation:

```
    \code{\link{function}}: function in this package
    \code{\link[MASS]{abbey}}: function in another package
    \link[=dest]{name}: link to dest, but show name
    \code{\link[MASS:abbey]{name}}: link to function in another package, but show name.
    \linkS4class{abc}: link to an S4 class
```
    +  Vers le web:   

```
    \url{http://rstudio.com}
    \href{http://rstudio.com}{Rstudio}
    \email{hadley@@rstudio.com} (note the doubled @)
```


*  Listes:  
    + Listes ordonnées:  
    
  
```
    #' \enumerate{
    #'   \item First item
    #'   \item Second item
    #' }
    
```
   
*   +   Listes non ordonnées à puces:  
  
```
    #' \itemize{
    #'   \item First item
    #'   \item Second item
    #' }
```

*   +   Listes nommées:  
  
```
    #' \describe{
    #'   \item{One}{First item}
    #'   \item{Two}{Second item}
    #' }
```

*   Mathematiques:    

LaTeX standard(sans les extensions):

```
    \eqn{a + b}: inline eqution
    \deqn{a + b}: display (block) equation
```
##	Exercice: Commenter les fonctions du package fars


#	Données

De nombreux packages R sont conçus pour manipuler, visualiser et modéliser des données; il peut donc être judicieux d'inclure certaines données dans votre package. La principale raison pour laquelle la plupart des développeurs incluent des données dans leur package est la démonstration de l'utilisation des fonctions incluses dans le package avec les données incluses. La création d'un package comme moyen de distribution des données est également une méthode de plus en plus populaire. De plus, vous voudrez peut-être inclure des données que votre package utilise en interne, mais qui ne sont pas disponibles pour quelqu'un qui utilise votre package. Lorsque vous incluez des données dans votre package, tenez compte du fait que votre fichier de package compressé doit être inférieur à 5 Mo, ce qui correspond à la taille de package maximale autorisée par CRAN. Si votre package dépasse 5 Mo, assurez-vous d'informer les utilisateurs dans les instructions de téléchargement et d'installation de votre package.

## Utiliser les données pour les demonstrations

### Objets

L'inclusion de données dans votre package est facile grâce au package devtools. Pour inclure des ensembles de données dans un package, créez d'abord les objets que vous souhaitez inclure dans votre package dans l'environnement global. Vous pouvez inclure n'importe quel objet R dans un package, pas seulement des trames de données. Assurez-vous ensuite que vous êtes dans le répertoire de votre package et utilisez la fonction use_data(), en répertoriant chaque objet que vous souhaitez inclure dans votre package. Les noms des objets que vous transmettez en tant qu’arguments à use_data() seront les noms des objets lorsqu’un utilisateur chargera le package. Assurez-vous donc que vous aimez les noms de variables que vous utilisez.

Vous devez ensuite documenter chaque objet de données que vous incluez dans le package. De cette manière, les utilisateurs du package peuvent utiliser la syntaxe d'aide courante de R, telle que ?Dataset pour obtenir plus d'informations sur le jeu de données inclus. Vous devez créer un fichier R appelé data.R dans le répertoire R de votre package. Vous pouvez écrire la documentation de données dans le fichier data.R. Jetons un coup d’œil à quelques exemples de documentation du packageage minimap. Nous allons d’abord regarder la documentation d’un bloc de données appelé maple:

```
#' Production and farm value of maple products in Canada
#'
#' @source Statistics Canada. Table 001-0008 - Production and farm value of
#'  maple products, annual. \url{http://www5.statcan.gc.ca/cansim/}
#' @format A data frame with columns:
#' \describe{
#'  \item{Year}{A value between 1924 and 2015.}
#'  \item{Syrup}{Maple products expressed as syrup, total in thousands of gallons.}
#'  \item{CAD}{Gross value of maple products in thousands of Canadian dollars.}
#'  \item{Region}{Postal code abbreviation for territory or province.}
#' }
#' @examples
#' \dontrun{
#'  maple
#' }
"maple"

```
Les data frame que vous incluez dans votre package doivent respecter le schéma général ci-dessus, dans lequel la page de documentation présente les attributs suivants:

Un titre décrivant l'objet.

Une balise @source décrivant où les données ont été trouvées.

Une balise @format qui décrit les données dans chaque colonne du cadre de données.

Et puis finalement une chaîne avec le nom de l'objet.

Le package de la mini-carte inclut également quelques vecteurs. Regardons la documentation de mexico_abb:

```
#' Postal Abbreviations for Mexico
#'
#' @examples
#' \dontrun{
#'  mexico_abb
#' }
"mexico_abb"
```

Vous devez toujours inclure un titre pour décrire un vecteur ou tout autre objet. Si vous devez détailler un vecteur, vous pouvez inclure une description dans la documentation ou une balise @source. Tout comme avec les data frame, la documentation d'un vecteur doit se terminer par une chaîne contenant le nom de l'objet.

### Données brutes

Une tâche courante pour les packages R consiste à extraire des données brutes de fichiers et à les importer dans des objets R afin de pouvoir les analyser. Vous souhaiterez peut-être inclure des exemples de fichiers de données brutes afin de pouvoir afficher différentes méthodes et options pour importer les données. Pour inclure des fichiers de données brutes dans votre package, vous devez créer un répertoire sous inst / extdata dans votre package R. Si vous avez stocké un fichier de données dans ce répertoire appelé response.json dans inst / extdata et que votre package s'appelle mypackage, un utilisateur peut accéder au chemin de ce fichier à l'aide de system.file ("extdata", "response.json", package = "mypackage"). Incluez cette ligne de code dans la documentation de votre package pour que vos utilisateurs sachent comment accéder au fichier de données brutes.

### Données accessibles en interne uniquement

Les fonctions de votre package peuvent avoir besoin d’avoir accès à des données auxquelles vous ne souhaitez pas que vos utilisateurs aient accès. Par exemple, le package swirl contient des traductions d’éléments de menu dans des langues autres que l’anglais, mais ces données n’ont rien à voir avec le but du package swirl et sont donc cachées de l’utilisateur. Pour ajouter des données internes à votre package, vous pouvez utiliser la fonction use_data() de devtools, mais vous devez spécifier l'argument internal = TRUE. Tous les objets que vous transmettez à use_data (..., internal = TRUE) peuvent être référencés sous le même nom dans votre package R. Tous ces objets seront enregistrés dans un fichier appelé R/sysdata.rda.

## Utiliser les packages pour partager les données

Plusieurs packages ont été créés dans le seul but de distribuer des données, notamment janeaustenr, gapminder, babynames et lego. L'utilisation d'un package R comme moyen de distribution des données présente des avantages et des inconvénients. D'une part, les données sont extrêmement faciles à charger dans R, un utilisateur n'a plus qu'à installer et charger le package. Cela peut être utile pour enseigner aux personnes qui débutent dans R et qui ne sont peut-être pas familiarisées avec l'importation et le nettoyage des données. Les packages de données vous permettent également de documenter des ensembles de données en utilisant roxygen2, qui fournit un livre de codes beaucoup plus propre et plus convivial pour les programmeurs par rapport à l’inclusion d’un fichier décrivant les données. D'autre part, les données d'un package de données ne sont pas accessibles aux personnes qui n'utilisent pas R, bien que rien ne vous empêche de les distribuer de plusieurs façons.

Si vous décidez de créer un package de données, vous devez documenter le processus que vous avez utilisé pour obtenir, nettoyer et enregistrer les données. Une approche pour cela consiste à utiliser la fonction use_data_raw() de devtools. Cela créera un répertoire à l'intérieur de votre package appelé data_raw. À l'intérieur de ce répertoire, vous devez inclure tous les fichiers bruts dont sont dérivés les objets de données de votre package. Vous devez également inclure un ou plusieurs scripts R qui importent, nettoient et enregistrent ces objets de données dans votre package R. Théoriquement, si vous aviez besoin de mettre à jour le package de données avec de nouveaux fichiers de données, vous devriez pouvoir simplement réexécuter ces scripts pour reconstruire votre package.

##	Exercice: Partager les données du package fars


#	Test du package

Une fois que vous avez écrit le code d’un package R et que vous pensez que cela fonctionne, c’est peut-être le bon moment pour prendre du recul et examiner quelques éléments de votre code.

*  Comment savez-vous que cela fonctionne? Étant donné que vous avez écrit les fonctions, vous avez certaines attentes quant à la manière dont elles doivent se comporter. Spécifiquement, pour un ensemble d'entrées donné, vous attendez une sortie donnée. Avoir ces attentes clairement en tête est un aspect important pour savoir si le code «fonctionne».  
*  Avez-vous déjà testé votre code? Tout au long du développement de votre code, vous avez probablement effectué de petits tests pour voir si vos fonctions fonctionnaient. En supposant que ces tests soient valides pour le code que vous testiez, il vaut la peine de les garder sous la main et de les intégrer à votre package.

La mise en place d’une batterie de tests pour le code de votre package peut jouer un rôle important dans le maintien du bon fonctionnement continu du package afin de rechercher les éventuels bug dans le code, le cas échéant. Au fil du temps, de nombreux aspects d’un package peuvent changer. Plus précisément:  

*  En développant activement votre code, vous pouvez modifier / supprimer l'ancien code sans le savoir. Par exemple, modifier une fonction sur laquelle de nombreuses autres fonctions s'appuient. Sans une structure de test complète, vous pourriez ne pas savoir qu'un comportement est rompu jusqu'à ce qu'un utilisateur vous le signale: Une fonction, un test.  
*  L'environnement dans lequel votre package s'exécute peut changer. La version de R peut changer, les libraries, les sites Web et toutes autres ressources externes, ainsi que les packages peuvent tous changer sans préavis. Dans de tels cas, votre code peut rester inchangé, mais en raison d'une modification externe, votre code peut ne pas produire la sortie attendue avec un ensemble d'inputs. Avoir des tests en place qui sont exécutés régulièrement peut aider à détecter ces changements, même si votre package n'est pas en développement actif.  
*  Lorsque vous corrigez des bugs dans votre code, il est souvent judicieux d’inclure un test spécifique qui corrige chaque bug afin d’être sûr que le bug ne se retrouvera pas dans une version ultérieure du package.  
    
Tester efficacement votre code a certaines implications dans la conception du code. En particulier, il peut être plus utile de diviser votre code en de plus petites fonctions pour pouvoir tester des morceaux individuels plus efficacement. Par exemple, si vous avez une grande fonction qui renvoie VRAI ou FAUX, il est facile de tester cette fonction, mais il ne sera peut-être pas possible d'identifier les problèmes au plus profond du code en vérifiant simplement si la fonction renvoie la valeur logique correcte. Il peut être préférable de diviser une fonction importante en fonctions plus petites afin de pouvoir tester séparément les éléments centraux de la fonction afin de s’assurer qu’ils se comportent correctement.

##	Package testthat

Le package testthat est conçu pour faciliter l'installation d'une batterie de tests pour votre package R. 

Le package contient essentiellement une suite de fonctions permettant de tester la sortie des fonctions. L'utilisation la plus simple du package est de tester une expression simple:


```r
library(testthat)
expect_that(sqrt(3) * sqrt(3), equals(3))
```

La fonction expect_that() peut être utilisée pour réaliser de nombreux types de tests, au-delà d'une simple sortie numérique. Le tableau ci-dessous fournit un bref résumé des types de comparaisons pouvant être effectuées.



```r
Fonction_test<-data.frame(
"Fonction de test"=c("equals()",
                     "is_identical_to()",
                     "is_equivalent_to()",
                     "is_a()",
                     "matches()", 
                     "prints_text()", 
                     "shows_message()",
                     "give_warning()" ,
                     "throws_error()",
                     "is_true()"),
Description=c("vérifie l'égalité",
              "vérifie l'égalité stricte",
              "comme equals() mais ignore les attributs d'objet",
              " vérifie la classe d'un objet (à l'aide de inherits())",
              "vérifie qu'une chaîne correspond à une expression régulière",
              "vérifie qu'une expression est imprimée sur la console",
              "vérifie si un message est généré",
              "vérifie qu'une expression donne un avertissement",
              "vérifie qu'une expression génère (correctement) une erreur",
              "vérifie qu'une expression est vraie"
)
)
knitr::kable(Fonction_test)
```



Fonction.de.test     Description                                                 
-------------------  ------------------------------------------------------------
equals()             vérifie l'égalité                                           
is_identical_to()    vérifie l'égalité stricte                                   
is_equivalent_to()   comme equals() mais ignore les attributs d'objet            
is_a()               vérifie la classe d'un objet (à l'aide de inherits())       
matches()            vérifie qu'une chaîne correspond à une expression régulière 
prints_text()        vérifie qu'une expression est imprimée sur la console       
shows_message()      vérifie si un message est généré                            
give_warning()       vérifie qu'une expression donne un avertissement            
throws_error()       vérifie qu'une expression génère (correctement) une erreur  
is_true()            vérifie qu'une expression est vraie                         

Un ensemble d’appels à expect_that() peuvent être regroupés avec la fonction test_that(), comme dans:



```r
test_that("model fitting", {
        data(airquality)
        fit <- lm(Ozone ~ Wind, data = airquality)
        expect_that(fit, is_a("lm"))
        expect_that(1 + 1, equals(2))
})
```

En règle générale, vous placeriez vos tests dans un fichier R. Si vous avez plusieurs ensembles de tests qui testent différents domaines d'un package, vous pouvez les placer dans des fichiers différents. Des fichiers individuels peuvent avoir leurs tests exécutés avec la fonction test_file(). Une collection de fichiers de test peut être placée dans un répertoire et testée avec la fonction test_di ().

Dans le contexte d'un package R, il est judicieux de placer les fichiers de test dans le répertoire tests. Ainsi, lors de l’exécution tous les tests seront exécutés dans le cadre du processus de vérification du package. Si l'un de vos tests échoue, le processus de vérification du package dans son intégralité échouera et vous empêchera de distribuer du code erroné. Si vous voulez que les utilisateurs puissent facilement voir les tests d'un package installé, vous pouvez placer les tests dans le répertoire inst / tests.

Vous pouvez trouvez plus d'aide sur ce package dans un article de Hadley Wickham paru dans le R Journal en 2011: "testthat: Get Started with Testing" [^8]. 



##	Soumettre au CRAN

Avant de soumettre un package à CRAN, vous devez réussir une batterie de tests exécutés R lui-même. Dans RStudio, si vous êtes dans un «projet» de package R, vous pouvez exécuter la vérification en cliquant sur le bouton Check de l'onglet Build. Cela va exécuter une série de tests qui vérifient les métadonnées de votre package, le fichier NAMESPACE, le code, la documentation, tous les tests, créer des vignettes et bien d’autres.

##	Exercice: Tester les fonctions du package fars

#	Open Source

##	Les licences
Vous pouvez spécifier la licnece de votre package dans le fichier DESCRIPTION du package. 
Le type de licence de votre package R est important car il fournit un ensemble de contraintes sur la manière dont les autres développeurs R vont pouvoir utiliser votre code. 
Si vous écrivez un package R à utiliser en interne dans votre société, celle-ci peut choisir de ne pas partager le package. Dans ce cas, octroyer une licence à votre package R est moins important car celui-ci appartient à votre société et n'est distribué qu'en interne. 

Dans le fichier DESCRIPTION, vous pouvez spécifier l'existence d'un fichier LICENSE, puis créer un fichier texte appelé LICENSE qui explique que votre société se réserve tous les droits sur le package.

Toutefois, si vous (ou votre entreprise) souhaitez partager publiquement votre package R, vous devez envisager l'octroi de licences open source. La philosophie de l'open source s'articule autour de trois principes:

*  Le code source du logiciel peut être inspecté.
*  Le code source du logiciel peut être modifié.
*  Les versions modifiées du logiciel peuvent être redistribuées.

Les licences open source offrent les protections ci-dessus. Parmi les licences open source, 3 sont plus populaires:  

*  GPL: General Public License (ou Licence publique générale GNU)
    > C'est une licence de logiciel libre et open source, **copyleft**, permettant donc d'inclure des modifications qu'à la condition qu'elles soient sous licence GPL.   
*  MIT: Massachusetts Institute of Technology
    > C'est une licence de logiciel libre et open source, **non copyleft**, permettant donc d'inclure des modifications sous d'autres licences, y compris non libres. 
    
*  CCO: Creative Commons
    > Les licences Creative Commons sont généralement utilisées pour des œuvres artistiques et créatives. C'est une licence **non copyleft** qui placera votre package dans le domaine public et qui n'obligera pas les utilisateurs à vous citer.
    
![Schéma de la réciprocité des logiciels libres issu de "Comprendre les licences de logiciels libres"[^9]](Images/Schema_OpenSource.PNG) 

##	Pourquoi l'open source

Vous avez mis des semaines à écrire un nouveau package R, alors pourquoi devriez-vous fournir une licence open source pour un logiciel que vous ou votre entreprise possédez par défaut? Pourquoi l’open sour source est une bonne idée?


*  **La communauté et la loi de Linus**  
Le développement de logiciels a commencé dans les milieux universitaires et les premiers programmes informatiques contenant du code pouvant être partagés et exécutés sur plusieurs ordinateurs étaient partagés entre les universitaires de la même manière que les universitaires partagent d'autres types de découvertes scientifiques. Le langage de programmation R est open source et il existe des centaines de packages R de haute qualité qui sont également open source. Un langage de programmation peut avoir beaucoup de fonctionnalités intéressantes mais la croissance et l'amélioration continues d'un langage sont rendues possibles par les personnes qui contribuent au logiciel écrit dans ce langage. 

**Pourquoi devriez-vous ouvrir le logiciel source?** 
Une des grandes raisons est un concept appelé **loi de Linus** qui fait référence à Linus Torvalds, le créateur de Linux. Le système d'exploitation Linux est un énorme projet de logiciel open source impliquant des milliers de personnes. Linux a une réputation de sécurité et de manque de bogues, ce qui est en partie dû au fait que tant de gens regardent et sont capables de modifier le code source. Si les utilisateurs de votre logiciel sont en mesure de visualiser et de modifier le code source de votre package R, celui-ci sera probablement amélioré en raison de la loi de Linus.

* **L'Open source est vecteur d'embauche**
    +   Si vous ouvrez votre logiciel open source et que d’autres personnes vous envoient des améliorations et des contributions, vous pouvez potentiellement identifier des candidats qui connaissent déjà votre code source. 
    +   si vous recherchez un emploi, vos contributions aux logiciels open source mettent en valeur vos compétences en logiciels.

#	Partage, contrôle de version et Github

**GitHub** vous permet de publier votre code en ligne dans des répertoires où tous les repertoires sont sous contrôle de version **git**. 

Vous pouvez publier des packages R sur GitHub et, avec la fonction install_github du package devtools, installer les packages R directement à partir de GitHub. 

GitHub peut être particulièrement utile pour collaborer avec d’autres sur les packages R, car il permet à tous les collaborateurs de transférer et de extraire du code entre leurs ordinateurs personnels et un répertoire GitHub. 

RStudio dispose de nombreuses fonctionnalités facilitant l’interface directe avec Git et GitHub.

Lorsque vous utilisez git et GitHub, 3 tâches sont essentielles:

1.  Configuration initiale - Vous ne le ferez qu'une fois  
    *  Télécharger git
    *  Configurez git avec votre nom d'utilisateur et votre email
    *  Configurer un compte GitHub
    *  Configurez une clé SSH pour relier RStudio sur votre ordinateur personnel à votre compte GitHub
    
2.  Configuration d’un repertoire spécifique:   
    c’est ce que vous devez faire chaque fois que vous créez un nouveau repertoire, mais vous ne devez le faire qu’une fois par repertoire.  
    *  Initialiser le répertoire sur votre ordinateur personnel en tant que repertoire git
    *  Effectuer une validation initiale des fichiers dans le repertoire
    *  Créer un repertoire GitHub vide
    *  Ajouter le repertoire GitHub en tant que branche distante du repertoire local
    *  Poussez le repertoire local vers la branche distante GitHub (Si vous partez d'un repertoire GitHub plutôt que d'un repertoire local, clonez le repertoire ou branchez-le et clonez-le à la place.)  
    
3.  Le flux de travail quotidien sur un repertoire - vous ferez cela régulièrement lors du développement du code dans un repertoire.  
    *  Valider les modifications dans les fichiers du repertoire pour enregistrer l'historique git localement: **commit**  
    *  Transmettre les modifications validées à la branche distante de GitHub: **push**  
    *  Extrayez la dernière version de la branche distante GitHub pour intégrer les modifications apportées par les collaborateurs au code de repertoire enregistré sur votre ordinateur: **pull**  
    *  Écrire et résoudre des «problèmes» avec le code dans le repertoire: **issue**  
    *  Résoudre les conflits de fusion qui surviennent entre les modifications de code effectuées par différents collaborateurs  

Le guide "Happy Git and GitHub for the useR"[^10] de Jenny Bryan est un excellent support dans la compréhension de l'utilisation de Git et Github sur R.


##	Git

Git est un système de contrôle de version. Lorsqu'un repertoire est sous contrôle de version git, les informations sur toutes les modifications apportées, enregistrées et validées sur tout fichier non ignoré d'un repertoire sont enregistrées. Cela vous permet de revenir aux versions précédentes du repertoire et de rechercher dans l'historique toutes les validations effectuées sur les fichiers suivis du repertoire. Si vous travaillez avec d'autres personnes, l'utilisation du contrôle de version git vous permet de voir toutes les modifications apportées au code, qui l'a fait et pourquoi (via les commits: messages de validation).

Vous aurez besoin de git sur votre ordinateur pour créer des repertoires git locaux que vous pouvez synchroniser avec les repertoires GitHub. Comme R, git est open source. Vous pouvez le [télécharger](https://git-scm.com/downloads) pour différents systèmes d'exploitation.

Après avoir téléchargé git mais avant de l’utiliser, vous devez le configurer. Par exemple, vous devez vous assurer que votre nom et votre adresse électronique sont indiqués. Vous pouvez configurer git à partir d'un shell bash (pour les Mac, vous pouvez utiliser «Terminal», tandis que pour les PC, vous pouvez utiliser GitBash, fourni avec l'installation de git).

Selon le système d'exploitation, l'installation diffère. Le mieux est de suivre le guide "Happy Git and GitHub for the useR"[^10].

Vous pouvez utiliser les fonctions de configuration de git pour configurer votre version de git. Deux modifications à apporter consistent à inclure votre nom et votre adresse électronique sous les noms user.name et user.email. Par exemple, le code suivant, s’il est exécuté dans un shell bash, configure un compte git pour un utilisateur nommé «Jane Doe» possédant une adresse électronique générique:

```
git config --global user.name "Jane Doe"
git config --global user.email "jane.doe@university.edu"
```

Une fois que vous avez installé git, vous devez redémarrer RStudio afin que RStudio puisse identifier que git est maintenant disponible. Souvent, le simple redémarrage de RStudio suffira. Cependant, dans certains cas, vous devrez peut-être prendre des mesures supplémentaires pour activer git dans RStudio. Pour ce faire, allez dans «RStudio» -> «Préférences» -> «Git / SVN». Choisissez «Activer le contrôle de version». Si RStudio ne trouve pas automatiquement votre version de git dans la case «Git exécutable» (vous le saurez si cette case est vide), recherchez le fichier exécutable git à l’aide du bouton «Parcourir» situé à côté de cette case. Si vous n’êtes pas sûr de l’emplacement de votre exécutable git, essayez d’ouvrir un shell bash et d’exécuter quel git, ce qui devrait vous donner le chemin du fichier si vous avez installé git.

### Initialiser un répertoire git

Vous pouvez initialiser un repertoire git en utilisant les commandes d'un shell bash ou directement depuis RStudio. 

1.  A partir d'un shell bash:

*  Utilisez un shell («Terminal» sur les Mac) pour accéder à ce répertoire. Vous pouvez utiliser cd pour le faire (similaire à setwd dans R).  
*  Une fois que vous êtes dans le répertoire, vérifiez d'abord qu'il ne s'agit pas déjà d'un repertoire git. Pour ce faire, lancez git status. Si vous obtenez le message fatal: pas un repertoire git (ni aucun des répertoires parent): .git, ce n'est pas encore un repertoire git. Si vous ne recevez pas d'erreur de la part de git status, le répertoire est déjà un repertoire, vous n'avez donc pas besoin de l'initialiser.
*  Si le répertoire n'est pas déjà un repertoire git, exécutez git init pour l'initialiser en tant que repertoire.

Par exemple, si je voulais créer un répertoire appelé «exemple_analyse», qui est un sous-répertoire direct de mon répertoire personnel, un repertoire git, je pourrais ouvrir un shell et exécuter:

```
cd ~/exemple_analyse
git init
```

2. A partir de R Studio.

*  Faites du répertoire un projet R. Si le répertoire est un package R, il contient probablement déjà un fichier .Rproj et il en va de même pour un projet R. Si le répertoire n'est pas un projet R, vous pouvez en créer un à partir de RStudio en allant dans «File» -> «New project» -> «Existing directory», puis accédez au répertoire dans lequel vous souhaitez créer un projet R.  
    +  Ouvrez le projet R.
    +  Allez dans «Tools» -> «Version control» -> «Project setup».  
    +  Dans la zone «Version control system», choisissez «Git».  

Si vous ne voyez pas «Git» dans la zone «Système de contrôle de version», cela signifie soit que Git n'est pas installé sur votre ordinateur, soit que RStudio n'a pas réussi à le trouver. Si tel est le cas, consultez les instructions précédentes pour vous assurer que RStudio a identifié l'exécutable de git.

Une fois que vous avez initialisé le projet en tant que repertoire git, vous devez avoir une fenêtre «Git» dans l’un de vos volets RStudio (volet supérieur droit par défaut). Lorsque vous apportez et sauvegardez des modifications dans des fichiers, elles apparaîtront dans cette fenêtre pour que vous puissiez les valider.  

![](Images/Git.png)

### Commit

Lorsque vous voulez que git enregistre les modifications, vous réalisez un **commit** ("engager la modification") permettant de prendre en compte les fichiers avec les modifications. Chaque fois que vous validez, vous devez inclure un court message de validation contenant des informations sur les modifications. Vous pouvez faire des commits depuis un shell. Cependant, le flux de travail le plus simple pour un projet R, y compris les packages R, consiste à effectuer des commits git directement à partir de l'environnement RStudio.

Pour effectuer une validation de RStudio, cliquez sur le bouton «Valider» dans la fenêtre Git. 

![](Images/confirm_commit.png)

Dans cette fenêtre, pour valider les modifications:  

1.  Cliquez sur les cases correspondant aux noms de fichiers dans le panneau en haut à gauche pour sélectionner les fichiers à valider.  
2.  Si vous le souhaitez, vous pouvez utiliser la partie inférieure de la fenêtre pour examiner les modifications que vous avez effectuées dans chaque fichier.  
3.  Écrivez un message dans la case «Valider le message» dans le panneau supérieur droit. Gardez le message sur une ligne dans cette case si vous le pouvez. Si vous devez en expliquer davantage, écrivez un court message d'une ligne, sautez une ligne, puis rédigez une explication plus longue.  
4.  Cliquez sur le bouton «Commit» à droite.

Une fois que vous avez validé les modifications apportées aux fichiers, ceux-ci disparaissent de la fenêtre Git jusqu'à ce que vous apportiez et sauvegardiez davantage de modifications.

### Naviguer dans l'historique

En haut à gauche de la fenêtre de validation, vous pouvez basculer sur «History». Cette fenêtre vous permet d’explorer l’historique des validations pour le repertoire. 

![](Images/Historique_git.png)

##	Github

### Cloner/Lier un répertoire local à Github

GitHub vous permet d’héberger des dépôts git en ligne. Cela vous permet de:

*  Travailler en collaboration sur un repertoire partagé  
*  Créez à partir du repertoire de quelqu'un d'autre votre propre copie que vous pourrez utiliser et modifier à votre guise  
*  Proposer des modifications aux repertoires d’autres personnes par le biais de demandes de tirage  

Pour ce faire, vous aurez besoin d’un compte GitHub. Vous pouvez vous inscrire sur [](https://github.com). Sur un compte gratuit, tous vos repertoires sont «publics» (visibles par tous).

Vous devez modifier certains paramètres dans RStudio afin que GitHub reconnaisse qu'il est possible de faire confiance à votre ordinateur, plutôt que de vous demander votre mot de passe à chaque fois. Pour ce faire, ajoutez une clé SSH de RStudio à votre compte GitHub en procédant comme suit:

1.  Dans RStudio, allez dans «RStudio» ->“Preferences” -> “Git / svn”. Choisissez “Create RSA key”.  
2.  Cliquez sur “View public key”. Copiez tout ce qui apparaît.  
3.  Accédez à votre compte GitHub et accédez à «Settings». Cliquez sur «SSH and GPG keys».  
4.  Cliquez sur «New SSH key». Nommez la clé quelque chose comme «Ordi Arnaud». Collez votre clé publique dans la «Key box».  

### 3 façons de connecter un répertoire à Github:

*  **Nouveau projet, Github en premier**  
*  **Projet existant, GitHub en premier**   
*  **Projet existant, GitHub en dernier**  

#### Nouveau projet, Github en premier

1.  Faire un repo sur GitHub  

    > Faites cela une fois par nouveau projet .

    >Aller à https://github.com et assurez-vous que vous êtes connecté.

    >Cliquez sur le bouton vert «New repository» . 
      *  Nom du répertoire : myrepo (ou ce que vous voulez)
      *  Public
      *  OUI initialisez ce répertoire avec un fichier README

    >Cliquez sur le gros bouton vert « Créer un répertoire».

    >Copiez l’URL de clone HTTPS dans votre presse-papiers via le bouton vert «Clone or download» . Ou copiez l'URL SSH si vous avez choisi de configurer des clés SSH.

2. Nouveau projet RStudio via git clone

Dans RStudio , démarrez un nouveau projet:

    >File > New Project > Version Control > Git . Dans «repository URL», collez l'URL de votre nouveau répertoire GitHub. Ça va être quelque chose comme ça https://github.com/arnaudmilet/myrepo.git .  
    >Bien choisir et se rappeler le chemin vers le projet.
    >Cliquez sur «Create Project» pour créer un nouveau répertoire, qui sera:  
        *  un répertoire ou "dossier" sur votre ordinateur  
        *  un répertoire Git, lié à un référentiel GitHub distant  
        *  un Projet RStudio   

3.  Apporter des modifications locales, enregistrer, commit

Faire ceci chaque fois que vous terminez un précieux morceau de travail , probablement plusieurs fois par jour .

De RStudio , modifiez le fichier README.md , par exemple, en ajoutant la ligne "Ceci est une ligne de RStudio”. Enregistrez vos modifications.

Validez ces modifications dans votre référentiel local:  

    >Cliquez sur l'onglet "Git" dans le volet supérieur droit.  
    >Cochez la case " Staged " pour tous les fichiers que vous voulez commiter.  
    >Tapez un message dans «Commit message», tel que «Commit from RStudio”.
    >Cliquez sur "Commit" 

4.  Poussez vos modifications locales vers GitHub (**Push**)

Faites cela quelques fois par jour.

Avant de faire un push, il est préférable de tirer depuis github (pull).Ca permet de prendre en compte les modifications faites par d'autres utilisateurs. 


5.  Confirmer le changement local

Regardez dans répertoire Github si les changement ont été pris en compte.


### Projet existant, GitHub en premier

Il s'agit ici de faire la même chose que précedemment puis de copier/coller un projet existant dans le nouveau projet et de le pousser vers Github.

### Projet existant, GitHub en dernier

1.  Vous avez un projet
2.  Faites en un repertoire Git, 3 possibiltés:
    *  usethis::use_git()
    *  Dans RStudio : Tools > Project Options … > Git/SVN. Dans “Version control system”, sélectionnez “Git”. Confirm New Git Repository? Yes!
    *  Dans un shell, dans le repertoire du projet faites un **git init**
3.  Continuez votre travail (modification, validation, commit,..)
4.  Connectez votre répertoire à Github avec usethis::use_github()



### Traitement des problèmes/bugs/modifications listés

Chaque répertoire GitHub original (c’est-à-dire qu’il ne s’agit pas d’une branche d’un autre répertoire) a un onglet intitulé «Issues». Cette page fonctionne comme un forum de discussion. Vous pouvez créer de nouveaux fils «Issue» pour décrire et discuter des choses que vous souhaitez modifier sur le répertoire.

Les problèmes peuvent être fermés une fois le problème résolu. Vous pouvez fermer les problèmes sur la page «Issues» avec le bouton «Close Issue». Si un commit que vous faites dans RStudio ferme un problème, vous pouvez le fermer automatiquement sur GitHub en incluant «Close # [numéro du problème]» dans votre message de commit, puis en le transmettant à GitHub. Par exemple, si le problème n°5 est «Corrigez une faute de frappe dans la section 3» et que vous apportez une modification pour corriger cette faute de frappe, vous pouvez créer et enregistrer la modification localement, puis prendre en compte cette modification avec le message de validation «Close #5», puis **push** (Pousser...) vers GitHub, et le problème n°5 dans "Issues" sera automatiquement fermé, avec un lien vers la validation qui corrigera le problème.

### Proposer des modifications sur le répertoire d'un autre utilsateur: "pull request"

Vous pouvez suggérer des modifications à un répertoire que vous ne possédez pas ou pour lequel vous n'avez pas l'autorisation de modifier directement. Procédez comme suit pour suggérer des modifications au répertoire de quelqu'un d’autre:


**1.  Forker le projet:**
Un fork est une copie d’un dépôt. Forker un dépôt vous permet d’expérimenter librement des modifications sans toucher au projet original.   
![](Images\Bouton_fork.png)

**2.  Créer une branche et travailler dessus**  
![](Images\Creer_branche.png)

**3.  Publier la branche sur son fork**  
**4.  Créer la pull-request**  

![[^11]](Images/fork-triangle-happy.png)

*  **Un dépôt de référence, conventionnellement appelé *upstream* **  
    C’est le dépôt du projet auquel nous voulons contribuer.  
    Nous n’avons que les droits en lecture dessus.  
*  **Un dépôt de fork, conventionnellement référencé *origin* **  
    C’est une copie du dépôt de référence.  
    Nous avons tous les droits dessus.  
*  **Un dépôt local**  
    C’est notre dépôt de travail.  

### Gestion des conflits

À un moment donné, si vous utilisez GitHub pour collaborer sur du code, vous obtiendrez des conflits. Cela se produit lorsque deux personnes ont modifié le même code de deux manières différentes en même temps.

Par exemple, supposons que deux personnes travaillent sur des versions locales du même répertoire et que la première personne modifie une ligne en mtcars [1,], tandis que la deuxième personne modifie la même ligne en tête (mtcars, 1). La deuxième personne insère ses commits dans la version GitHub du répertoire avant la première personne. Désormais, lorsque la première personne extraira les derniers commits dans le répertoire GitHub, il y aura un conflit pour cette ligne. Pour pouvoir statuer sur une version finale, la première personne devra décider quelle version du code utiliser et valider une version du fichier avec ce code.

S'il y a des conflits de fusion, ils apparaîtront comme ceci dans le fichier:


```r
<<<<<<< HEAD
mtcars[1, ]
=======
head(mtcars, 1)
>>>>>>> remote-branch
```

Pour les résoudre, recherchez tous ces endroits dans les fichiers en conflit, choisissez le code que vous souhaitez utiliser et supprimez tout le reste. Pour l'exemple de conflit, il pourrait être résolu en modifiant le fichier comme ceci:


```r
head(mtcars, 1)
```

Ce conflit est maintenant résolu. Une fois que vous avez résolu tous les conflits dans tous les fichiers du répertoire, vous pouvez enregistrer et valider les fichiers.

Ces conflits peuvent survenir dans quelques situations:

*  Vous recevez les commits de la branche GitHub du répertoire sur lequel vous avez travaillé localement.  
*  Quelqu'un fait une pull request pour l'un de vos répertoires et vous avez mis à jour une partie du code entre le moment où la personne a créé le répertoire et soumis la pull request.


##	Exercice: Déposer les packages monpackage et fars sur github


#	Philosophie et design du package
##	Philosophie Unix

Le langage de programmation R est un logiciel open source et de nombreux packages logiciels open source s’inspirent de la conception du système d’exploitation Unix sur lequel sont basés macOS et Linux. Ken Thompson - l'un des concepteurs d'Unix - a d'abord exposé cette philosophie, et de nombreux principes de philosophie Unix peuvent être appliqués aux programmes R. Le thème philosophique général des programmes Unix est de bien faire une chose. S'en tenir à cette règle remplit plusieurs objectifs:

*  Étant donné que votre programme ne **fait qu'une chose**, les chances que votre programme contienne plusieurs lignes de code sont réduites. Cela signifie que les autres peuvent plus facilement lire le code de votre programme afin qu’ils puissent comprendre exactement comment cela fonctionne (s’ils ont besoin de savoir).  
*  La simplicité de votre programme réduit les risques de problèmes majeurs, car **moins de lignes de code** signifient moins de risque de commettre une erreur: **lapply, purr: utiliser la vectorisation**  
*  Votre programme sera plus facile à comprendre pour les utilisateurs si les inputs et outputs sont réduits: **Une fonction ne fait qu’une chose.**    
*  Les programmes construits avec d'autres petits programmes ont plus de chance d'être également petits. Cette possibilité de **chaîner plusieurs petits programmes** pour en faire un programme plus complexe (mais également petit) est appelée "composabilité": Avec le package **magrittr**, les pipes en R sont facilités.   

##	Valeurs par défaut

Fournir un maximum de valeurs par défaut dans vos fonctions permet de limiter le risque d'inputs incorrects ou inattendus. Vous devez  que ce qui est raisonnable. 

##	Nommer les éléments

Nommer les fonctions et les variables pour que leur utilisation soit simple dans R:

*  Utilisez "_" et les minuscules. Les packages R modernes utilisent des noms de fonction et de variable comme geom_line (), bind_rows () et unnest_token (),...  
*  Utilisez des noms courts  
*  Les noms doivent être significatifs et descriptifs.  
*  Assurez-vous de ne pas attribuer des noms existants et communs dans R. Vous pouvez vérifier si un nom est pris à l'aide de la fonction apropos():  


```r
apropos("mean")
```

```
##  [1] ".colMeans"     ".rowMeans"     "colMeans"      "kmeans"       
##  [5] "mean"          "mean.Date"     "mean.default"  "mean.difftime"
##  [9] "mean.POSIXct"  "mean.POSIXlt"  "rowMeans"      "weighted.mean"
```

Vous pouvez envisager de regrouper des fonctions similaires dans des familles qui commencent toutes par le même préfixe court. Par exemple, dans le package ggplot2, la famille de fonctions aes_ définit l'esthétique des graphismes, la famille de fonctions gs_ interagit avec l'API Google Sheets du package googlesheets, la famille de fonctions geom_ initialise les graphiques, ...

##	Respecter la communauté d’utilisateurs

Si vous écrivez un package avec des fonctions utiles et bien conçues, vous aurez peut-être la chance que votre package devienne populaire! D'autres peuvent utiliser vos fonctions pour étendre ou adapter leurs fonctionnalités à d'autres fins. Cela signifie que lorsque vous établissez un ensemble d’arguments pour une fonction, vous promettez implicitement une certaine stabilité pour les inputs et les outputs de cette fonction. Changer l’ordre ou la nature des arguments de fonction ou des valeurs de retour peut casser le code d’autres personnes, ajouter du travail et causer du tort à ceux qui ont choisi d’utiliser votre logiciel.   
Si vous pensez que les fonctions d’un package que vous développez ne sont pas encore stables, vous devez en informer les utilisateurs afin qu’ils soient avertis s’ils décident de développer votre travail.

#	Intégration continue

Utilser des services d’intégration continue permet de réaliser des tests de vos packages pour des systèmes d'exploitation ne correspondant pas forcément au vôtre:   
*  **Travis**: Tester votre package sous Linux  
*  **AppVeyor**: Tester votre package sous Windows  

Ces deux services sont gratuits pour les packages R construits dans des répertoires GitHub. Ces services d'intégration continue seront exécutés chaque fois que vous réaliserez un push. Ces services s’intègrent parfaitement à GitHub afin que vous puissiez voir si votre package est construit correctement ou non.

##	Travis

Pour commencer à utiliser Travis, allez sur https://travis-ci.org et connectez-vous avec votre compte GitHub. En cliquant sur votre nom dans le coin supérieur droit du site, une liste de vos dépôts publics GitHub apparaîtra. 

Ouvrez votre console R et accédez à votre répertoire de packages R. Maintenant, chargez le package devtools avec la library(devtools) et entrez use_travis() dans votre console R. Cette commande va configurer un fichier .travis.yml de base pour votre package R. Vous pouvez maintenant ajouter, valider et appliquer vos modifications à GitHub, ce qui déclenchera la première construction de votre package sur Travis. Retournez à https://travis-ci.org pour voir votre package construit et testé en même temps! 

Une fois que votre package a été construit pour la première fois, vous pourrez obtenir un badge. Il s’agit d’une petite image générée par Travis qui indique si votre package est construit correctement et qu’il réussit tous vos tests. Vous pouvez afficher ce badge dans le fichier README.md du répertoire GitHub de votre package afin que vous et les autres utilisateurs puissiez contrôler l’état votre test sur Linux.

##	Appveyor

Vous pouvez commencer à utiliser AppVeyor en allant sur https://www.appveyor.com/ et en vous connectant avec votre compte GitHub. Une fois connecté, cliquez sur «Projects» dans la barre de navigation supérieure. Si vous avez des répertoires GitHub qui utilisent AppVeyor, vous pourrez les voir ici. Pour ajouter un nouveau projet, cliquez sur "Nouveau projet" et recherchez le répertoire GitHub correspondant au package R que vous souhaitez tester sous Windows. Cliquez sur «Add» pour qu'AppVeyor commence à suivre ce répertoire.

Ouvrez votre console R et accédez à votre répertoire de packages R. Maintenant, chargez le package devtools avec la library(devtools) et entrez use_appveyor() dans votre console R. Cette commande configurera un fichier appveyor.yml par défaut pour votre package R. Vous pouvez maintenant ajouter, valider et appliquer vos modifications à GitHub, ce qui déclenchera la première construction de votre package sur AppVeyor. 

Retournez à https://www.appveyor.com/ pour voir le résultat de la construction.

Comme Travis, AppVeyor génère également des badges que vous pouvez ajouter au fichier README.md du répertoire GitHub de votre package.

## Un mot sur les chemins d'accès

Les chemins d'accès aux fichiers et aux dossiers peuvent avoir de grandes différences entre les systèmes d'exploitation. En général, vous devriez éviter de créer un chemin "à la main". Par exemple, si je voulais accéder à un fichier appelé data.txt, il se trouverait sur le bureau de l'utilisateur à l'aide de la chaîne "~ / Desktop / data.txt". fonctionne si ce code a été exécuté sur une machine Windows. En général, vous devez toujours utiliser des fonctions pour créer et trouver des chemins d'accès aux fichiers et aux dossiers. La méthode de programmation correcte pour construire le chemin ci-dessus consiste à utiliser la fonction file.path(). Donc, pour obtenir le fichier ci-dessus, je voudrais faire ce qui suit:


```r
file.path("~", "Desktop", "data.txt")
```

```
## [1] "~/Desktop/data.txt"
```


En général, il n’est pas garanti sur un système que le fichier ou le dossier que vous recherchez existera. Toutefois, si l’utilisateur de votre package a installé votre package, vous pouvez être sûr que tous les fichiers de votre package existent sur leur ordinateur. Vous pouvez trouver le chemin des fichiers inclus dans votre package en utilisant la fonction system.file(). Tous les fichiers ou dossiers du répertoire inst/ de votre package seront copiés à partir du niveau suivant, une fois votre package installé. Si votre package s'appelle ggplyr2 et qu'il y a un fichier sous inst/data/first.txt, vous pouvez obtenir le chemin de ce fichier avec system.file("data", "first.txt", package = "ggplyr2"). 


# Ressources utiles
[^1]: [Writing R Extensions](https://cran.r-project.org/doc/manuals/r-release/R-exts.html)
[^2]: [Hadley Wickham, 2015, R packages, O’Reilly](http://r-pkgs.had.co.nz/)
[^3]: [Roger D. Peng, Sean Kross, and Brooke Anderson, 2017, Mastering Software Development in R](https://bookdown.org/rdpeng/RProgDA/)
[^4]: [ThinkR, Vincent Guyader, Fabriquer un package R en quelques minutes](https://thinkr.fr/creer-package-r-quelques-minutes/)
[^5]: [R Markdown: The Definitive Guide](https://bookdown.org/yihui/rmarkdown/)
[^6]: [R Markdown CheatSheet](https://www.rstudio.com/resources/cheatsheets/#rmarkdown)
[^7]: [Hadley Wickham, 2018, Vignette roxygen2 : "Text formatting reference sheet""](https://cran.r-project.org/web/packages/roxygen2/vignettes/formatting.html)
[^8]: [Hadley Wickham, 2011, R Journal, "testthat: Get Started with Testing"](https://journal.r-project.org/archive/2011-1/RJournal_2011-1_Wickham.pdf)
[^9]: [Robert Viseur, 2014, Présentation dans le cadre des jeudis du libre à l'UMONS, Comprendre les licences de logiciels libres](https://www.slideshare.net/ecocentric/rv-jdllicences20140220online?from_action=save)
[^10]: [Jenny Bryan, the STAT 545 TAs, Jim Hester, Happy Git and GitHub for the useR](https://happygitwithr.com/)
[^11]: [Les pull-request : comment ça marche, comment en faire une, comment en intégrer une ?](https://blog.zenika.com/2017/01/24/pull-request-demystifie/)
