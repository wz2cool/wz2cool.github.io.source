---
title: 2.0新增DynamicQuery类
date: 2017-08-11 17:51:55
tags:
- java
- MDQ 2.0 杂记
---
新增加一个DynamicQuery类，类似于tk.mybatis.mapper中的Example，这样我们可以把一些属性封装到一个类中方便以后扩展。 这里目前封装了筛选，排序和一个distinct。
```java
public class DynamicQuery<T> {
    private boolean distinct;
    private final Class<T> entityClass;
    private final List<FilterDescriptorBase> filters = new ArrayList<>();
    private final List<SortDescriptor> sorts = new ArrayList<>();

    public DynamicQuery(Class<T> entityClass) {
        this.entityClass = entityClass;
    }

    public Class<T> getEntityClass() {
        return entityClass;
    }

    public FilterDescriptorBase[] getFilters() {
        return filters.toArray(new FilterDescriptorBase[filters.size()]);
    }

    public SortDescriptor[] getSorts() {
        return sorts.toArray(new SortDescriptor[sorts.size()]);
    }

    public boolean addFilter(FilterDescriptorBase filter) {
        return this.filters.add(filter);
    }

    public boolean removeFilter(FilterDescriptorBase filter) {
        return this.filters.remove(filter);
    }

    public boolean addSort(SortDescriptor sort) {
        return this.sorts.add(sort);
    }

    public boolean removeSort(SortDescriptor sort) {
        return this.sorts.remove(sort);
    }

    public boolean isDistinct() {
        return distinct;
    }

    public void setDistinct(boolean distinct) {
        this.distinct = distinct;
    }
}
```