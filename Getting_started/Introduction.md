# 歴史

CloudCompare は 3D ポイントクラウド (および三角形メッシュ) を編集し、処理するためのソフトウェアです。

もともとは、高密度の3次元ポイントクラウド同士の直接比較に性能が出るようデザインされました。それは明らかに、この種のタスクを実行する時に非常に効果的な[octree](http://www.cloudcompare.org/doc/wiki/index.php?title=Octree)に依拠しています [^1]。それに加えて、CloudCompareは地上式の[レーザースキャナー](http://en.wikipedia.org/wiki/Laser_scanning)から得られるなどした、巨大なポイントクラウド (典型的には2005年(!)の時点で1,000万ポイント以上) を スタンダードなノートパソコンで扱えることを意味します。まもなく、ポイントクラウドと三角形メッシュの比較機能がサポートされるようになります (以下、参照)。その後は、他の多くのポイントクラウド処理のアルゴリズム (registration、リサンプリング、color/normal vectors/scalar fields management、統計的計算、センサーマネジメント、interactive or automatic segmentation等々) がディスプレイ拡張ツール (カスタムカラーランプ、color & normal vectors handling、calibrated pictures handling、OpenGL shaders、プラグイン等々) と同様に使えるようになるでしょう。

![http://www.cloudcompare.org/doc/wiki/images/9/98/Comp_cloud_mesh.jpg](http://www.cloudcompare.org/doc/wiki/images/9/98/Comp_cloud_mesh.jpg)

[^1]: たとえば、デュアルコアプロセッサーのノートパソコンで300万ポイントの距離を14,000の三角形メッシュと計算するのに10秒しかからない。

# 哲学
## ポイントクラウド対メッシュ

特にその歴史において、CloudCompareは、ほぼすべての3Dエンティティは[ポイントクラウド]( http://www.cloudcompare.org/doc/wiki/index.php?title=Entities )だと考えています。特徴的なものは、 a triangular mesh is only a point cloud (the mesh vertices) with an associated topology (triplets of 'connected' points corresponding to each triangle). This explains that meshes have always either a point cloud named 'vertices' as sibling or parent  (depending on the way they have been loaded or generated). And while CloudCompare will let the user apply some tools directly on a mesh structure (i.e. triangles), some tools can only be applied to the mesh vertices. It may be a bit disturbing at first, but we don't want the user to ignore this: '''CloudCompare is mainly a point cloud processing software'''.

Of course, as CloudCompare is meant to do change detection (e.g. subsidence monitoring) and as a triangular mesh is a very common way to represent a reference shape (e.g. a building), it is very useful and it couldn't be ignored. Nevertheless it remains a "secondary" entity, especially as CloudCompare is able to compare two point clouds directly, without the need to generate an intermediary mesh.

The main reasons for this are:
* meshes are generally very hard to generate properly on real-life scenes, especially when scanned with a laser scanner (noise, variable density, etc.)
* and as ALS/TLS point clouds are generally very dense (and accurate), we already have all the information we need


## Scalar fields

Among all 'features' that can be associated to a point cloud (colors, normals, etc.) one has a particular place in CloudCompare: the ''[[Entities | scalar field]]''.

A scalar field is simply a set of values (one per point - e.g. the distance of each point to another entity). As each value is associated to a point (or vertex) it is possible to display those values as colors (with custom color ramps) or to apply filters on them (smooth, gradient, etc.), some basic math operations (exp, log, power of 2 or 3, cos, sin, tan, etc.) and of course to segment the cloud relatively to those values (thresholding, local statistical filtering, etc.).

CloudCompare can handle multiple scalar fields on the same cloud. It is even possible to apply simple arithmetic operations (-,+,/,*) between two scalar fields of a same cloud.

# Some technical considerations

## Portable

CloudCompare is developed in C++. It is currently compiled on Windows, Linux and Mac OS (thanks to [http://www.cmake.org/ C-make]) and for 32bits and 64bits architectures.


## Trade-off between storage and speed

Here are some details about the technical choices that have been made in CloudCompare (mainly to achieve the goal of loading as much points as possible without downgrading too much performances - i.e. a good trade-off between storage and speed):
* all stored values and most of computations are done with [http://en.wikipedia.org/wiki/Single-precision_floating-point_format 32bits floating-point values]
* to prevent any limitation on the size of arrays (as it's hard to get a big contiguous block of memory on Windows 32 bits), we use a custom container that automatically chunks datasets in small blocks (64 Kb per block).
* normal vectors (if any) are compressed on 16 bits (15 bits actually, because of the way [http://en.wikipedia.org/wiki/Quantization quantization] works)
* the specific [[octree]] structure used in CloudCompare requires constant per-point memory (i.e. 8 bytes per point on a 32 bits OS - with a maximum depth of 10 - and 12 bytes on a 64 bits OS - with a maximum depth of 21!). It is based on a particular quantization of the 3D point coordinates - a kind of [http://en.wikipedia.org/wiki/Z-order_curve Morton] ordering scheme - where each point position in the octree grid and at any level is represented by a single integer code. We then process those codes to achieve very efficient nearest-neighbors querying operations. However, while this octree structure is very efficient for computing distances for instance, it's not suitable for fast display (Level Of Detail, etc.).

The result of the above choices is that CloudCompare can store about 90 million blank points per gigabyte of memory. If you add RGB colors, normal vectors, a single scalar field and if you need to compute the octree, you can load up to 32 million points per gigabyte.

On a 64 bits OS you can load as many points as you want (''well, up to 4 billion in fact''). ''However, depending on your graphic card capabilities, display and interactivity may be severely downgraded with this many points'' ;). With a high-end graphic card you can keep a reasonable frame rate with up to 150 million points.

# Recent evolution

While the project has started in 2004 at [http://research.edf.com/research-and-innovation-44204.html EDF R&D], it has only been released in the public domain around 2009 (under GPL license). As CloudCompare is on open-source project, everyone is free (and welcome) to extend its capabilities. Don't hesitate to ask questions and share your experiences on the [http://www.cloudcompare.org/forum forum] and to take a look at the [https://github.com/cloudcompare/trunk Github] source repository.
