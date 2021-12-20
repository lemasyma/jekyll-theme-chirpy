---
title:          "CAMA : ma03 Matrice Camera"
date:           2020-03-30 10:00
categories:     [S6, Shannon, CAMA]
tags:           [S6, CAMA, Shannon]
math: true
description: Matrice Camera
---
Lien de la [note Hackmd](https://hackmd.io/@lemasymasa/rJDGOYDhL)
# Cours du 30 / 03

<div class="alert alert-info" role="alert" markdown="1">
Calculons l'image que genere une camera :
* positionnee en $(c_x, c_y, c_z)$
* regardant dans la direction $(v_x, v_y, v_z)$
* avec un angle de rotation  $c_\theta$ (que l'on prend = 0 pour commencer)
* avec une focale $f$
</div>

<div class="alert alert-danger" role="alert" markdown="1">
On a pour tout point $X$ de l'espace sa position $x$ sur l'image donnée par 

$$
x = P X
$$

* $P$ : matrice qui représente l'action de la caméra.
</div>
Le but est de trouver $P$.

## Dimensions

* $X$ : point en 3D
    * on rajoute une dimension pour les translations (cf ma02) : $X = (X_x, X_y, X_z, 1)$
* $x$ : point en 2D
    * pour les translation : $x = (x_x, x_y, 1)$
* $P$ matrice de dimensions $3*4$

## Repères
3 reperes : 
* celui du monde en en 3D
* celui de l'image en 2D
* celui de la camera en 3D

## Focale
<div class="alert alert-danger" role="alert" markdown="1">
On représente la focale comme la distance entre l'origine est la position virtuelle de l'image 2D.
</div>
![](https://i.imgur.com/FW5BVZV.png)

Dans le repere de la camera : $x = \frac{f}{X_z} X = f \frac{X}{X_z}$. 

Si on bouge uniquement la focale, et que le repere de la camera est le meme que celui du monde alors : 
$$\textrm{si }\quad P = 
\begin{bmatrix}
f & 0 & 0 & 0 \\
0 & f & 0 & 0 \\
0 & 0 & 1 & 0 \\
\end{bmatrix}
\quad \textrm{ on a }\quad
P X = 
\begin{bmatrix}
f X_x \\
f X_y \\
X_z \\
\end{bmatrix}
$$
<div class="alert alert-warning" role="alert" markdown="1">
C'est presque le resultat recherche, on a $x$ a un facteur $X_z$ pret.
</div>
<div class="alert alert-success" role="alert">
Pour garantir $x_z = 1$ on ajoute une normalisation : 
</div>
``` python
f = 0.5 # focale

F = lambda f: np.array([[f, 0, 0, 0], [0, f, 0, 0], [0, 0, 1, 0]])
```
```
F = array([[0.5, 0. , 0. , 0. ],
       [0. , 0.5, 0. , 0. ],
       [0. , 0. , 1. , 0. ]])
```
``` python
def normalize(x):
    x[0,:] /= x[2,:]
    x[1,:] /= x[2,:]
    return x[:2,:]
```

## Changement de repère
<div class="alert alert-danger" role="alert" markdown="1">
L'axe principal de la camera est $z$, soit $x$ dans le repere du monde 3D. On choisit comme repère inital de la caméra : $(x,y,z)_{cam} = (y, z, x)_{3D}$.
</div>
<div class="alert alert-info" role="alert" markdown="1">
La matrice de passage est:
$$ X_{cam} = 
\begin{bmatrix}
0 & 1 & 0 \\
0 & 0 & 1 \\
1 & 0 & 0 \\
\end{bmatrix}
\, X_{3D}
$$
</div>
Pour respecter la notation $(x, y, z, 1)$ : 
$$P = 
\begin{bmatrix}
f & 0 & 0 & 0 \\
0 & f & 0 & 0 \\
0 & 0 & 1 & 0 \\
\end{bmatrix}
\quad
\begin{bmatrix}
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
1 & 0 & 0 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
$$

## Translation de la caméra
<div class="alert alert-info" role="alert" markdown="1">
Si la camera est en $(c_x, c_y, c_z)$ et non en $(0, 0, 0)$, c'est une **translation :**

$$T = 
\begin{bmatrix}
1 & 0 & 0 & -c_x \\
0 & 1 & 0 & -c_y \\
0 & 0 & 1 & -c_z \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
$$
</div>

## Axe principal de la caméra
On change la direction de la camera et son axe principal n'est plus $x$ du monde 3D.
<div class="alert alert-danger" role="alert" markdown="1">
Pour pointer un vecteur 3D dans une direction, il faut 2 rotations autour de 2 axes orthogonaux a notre vecteur. En 2D il suffit d'une rotation autour de $z$.
</div>
Pour diriger la camera dans une direction $v$, les rotations se font autour des axes $z$ et $y$ du monde: 
* la rotation horizontale $\psi$ tourne autour de $z$
* la rotation verticale $\phi$ tourne autour de $y$

<div class="alert alert-info" role="alert" markdown="1">
$$D = 
\begin{bmatrix}
cos(\phi) & 0 & sin(\phi) & 0 \\
0 & 1 & 0 & 0 \\
-sin(\phi) & 0 & cos(\phi) & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
\;
\begin{bmatrix}
cos(\psi) & -sin(\psi) & 0 &  0 \\
sin(\psi) & cos(\psi) & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
$$
</div>
``` python
def D(ah, av):
    if type(ah) == int:
        ah = ah * 2 * np.pi / 360
        av = av * 2 * np.pi / 360
    rh = np.array([[np.cos(ah), -np.sin(ah), 0, 0], [np.sin(ah), np.cos(ah), 0, 0], [0, 0, 1, 0], [0, 0, 0, 1]])
    rv = np.array([[np.cos(av), 0, np.sin(av), 0], [0, 1, 0, 0], [-np.sin(av), 0, np.cos(av), 0], [0, 0, 0, 1]])
    return rv @ rh
```