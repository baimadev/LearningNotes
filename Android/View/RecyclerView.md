## 缓存机制

一般来说有四级缓存，但其实只有三级，还有一种为自定义缓存。

- Scrap（mAttachedScrap、mChangedScrap）
- mCachedViews
- mRecyclerPool

每次调用fill填充表项时，都会从缓存中依次取ViewHolder，如果都不符合条件就直接调用onCreateViewHolder和onBindViewHolder。

### Scrap

mAttachedScrap用于屏幕中可见表项的回收和复用。是直接复用，不用调用onCreateViewHolder和onBindViewHolder。

具体逻辑在LayoutManager中，每次布局都会调用detachAndScrapAttachedViews回收当前可见表项，然后再填充。

这样做是因为 RecyclerView 要做表项动画，需要比对动画前和动画后的两张“表项快照”，为了获得两张快照，就得布局两次，分别是预布局和后布局。  
Scrap缓存了这些表项，以便更快的填充，Scrap的生命周期为从布局开始到布局结束。


### mCachedViews

用于存储滑出屏幕的表项，mCachedViews是 ArrayList ，默认存储最多2个 ViewHolder，先进先出。  
当mCachedViews存满了后，会将最老的放进RecyclerPool中。

复用条件：mCachedViews是离屏缓存，用于缓存指定位置的 ViewHolder ，只有“列表回滚”这一种场景（刚滚出屏幕的表项再次进入屏幕），才有可能命中该缓存。
也不需要再绑定数据。


### mRecyclerPool

RecycledViewPool 对 ViewHolder 按viewType分类存储（通过SparseArray），同类 ViewHolder 存储在默认大小为5的ArrayList中。

从mRecyclerPool中复用的 ViewHolder ，只能复用于viewType相同的表项，且需要重新绑定数据，
