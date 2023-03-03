---
title: Java使用Criteria实现多条件查询
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-03-03 15:43:37
img:
coverImg:
password:
summary:
tags:
- mongodb
- spring
categories: [Programming Language,Java language,框架（Framework）,Spring]
---

# 问题概述
前端传入多个查询条件，根据查询条件、查询字段和值使用Java对MongoDB进行多条件筛选。

实现方法：
查询条件传入List<JSONObject>，格式如下：

```json
[
    {
        "condition": "is",
        "key":"name"
        "value": "xxxxxx"
    },
    {
        "condition": "gte",
        "key":"age"
        "value": 18
    }
]
```
# 自定义查询帮助类

```java
import com.alibaba.fastjson.JSONObject;
import org.apache.commons.lang3.ObjectUtils;
import org.apache.commons.lang3.StringUtils;
import org.bson.Document;
import org.springframework.data.domain.Sort;
import org.springframework.data.mongodb.core.query.BasicQuery;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
 
import java.util.*;
import java.util.regex.Pattern;
 
/**
 * 自定义查询帮助类
 *
 * @author xkk
 * create Date: 2019/11
 */
public class MongoUtil {
 
    /**
     * 等于
     */
    private static final String IS = "is";
 
    /**
     * 小于
     */
    private static final String LT = "lt";
 
    /**
     * 小于等于
     */
    private static final String LTE = "lte";
 
    /**
     * 大于
     */
    private static final String GT = "gt";
 
    /**
     * 大于等于
     */
    private static final String GTE = "gte";
 
    /**
     * in
     */
    private static final String IN = "in";
 
    /**
     * not in
     */
    private static final String NIN = "nin";
 
    /**
     * 并且
     */
    private static final String AND = "and";
 
    /**
     * 不等于
     */
    private static final String NE = "ne";
 
    /**
     * 数字不等于
     */
    private static final String NUM_NE = "num_ne";
 
    /**
     * 数组不等于
     */
    private static final String ARR_NE = "arr_ne";
 
    /**
     * all
     */
    private static final String ALL = "all";
 
    /**
     * all
     */
    private static final String LIKE = "like";
 
    /**
     * all
     */
    private static final String NUM_IS = "num";
 
    /**
     * key
     */
    private static final String KEY = "key";
 
    /**
     * value
     */
    private static final String VALUE = "value";
 
    /**
     * 正序
     */
    private static final String ASC = "asc";
 
    /**
     * 倒序
     */
    private static final String DESC = "desc";
 
    /**
     * 添加查询条件,生成ArrayList<Criteria>
     *
     * @param conditionList 条件列表
     * @return
     */
    private static Criteria[] genCriteria(List<JSONObject> conditionList) {
 
        ArrayList<Criteria> arrayList = new ArrayList<>();
 
        conditionList.forEach(object -> {
            Criteria criteria;
            // 查询条件
            String condition = object.getString("condition");
            if (StringUtils.isBlank(condition)) {
                return;
            }
            if (ObjectUtils.isEmpty(object.get(VALUE))) {
                return;
            }
            // 查询条件可根据需求进行扩展
            switch (condition) {
                case IS: {
                    criteria = Criteria.where(object.getString(KEY)).is(object.getString(VALUE));
                    arrayList.add(criteria);
                    break;
                }
                case LT: {
                    criteria = Criteria.where(object.getString(KEY)).lt(object.getInteger(VALUE));
                    arrayList.add(criteria);
                    break;
                }
                case LTE: {
                    criteria = Criteria.where(object.getString(KEY)).lte(object.getInteger(VALUE));
                    arrayList.add(criteria);
                    break;
                }
                case GT: {
                    criteria = Criteria.where(object.getString(KEY)).gt(object.getInteger(VALUE));
                    arrayList.add(criteria);
                    break;
                }
                case GTE: {
                    criteria = Criteria.where(object.getString(KEY)).gte(object.getInteger(VALUE));
                    arrayList.add(criteria);
                    break;
                }
                case IN: {
                    criteria = Criteria.where(object.getString(KEY)).in(object.getString(VALUE));
                    arrayList.add(criteria);
                    break;
                }
                case NIN: {
                    criteria = Criteria.where(object.getString(KEY)).nin(object.getString(VALUE));
                    arrayList.add(criteria);
                    break;
                }
                case AND: {
                    criteria = Criteria.where(object.getString(KEY)).and(object.getString(VALUE));
                    arrayList.add(criteria);
                    break;
                }
                case NUM_NE: {
                    criteria = Criteria.where(object.getString(KEY)).ne(object.getInteger(VALUE));
                    arrayList.add(criteria);
                    break;
                }
                case NE: {
                    criteria = Criteria.where(object.getString(KEY)).ne(object.get(VALUE));
                    arrayList.add(criteria);
                    break;
                }
                case ARR_NE: {
                    criteria = Criteria.where(object.getString(KEY)).ne(object.get(VALUE));
                    criteria = Criteria.where(object.getString(KEY)).elemMatch(criteria);
                    arrayList.add(criteria);
                    break;
                }
                case ALL: {
                    criteria = Criteria.where(object.getString(KEY)).all(object.getJSONArray(VALUE).toArray());
                    arrayList.add(criteria);
                    break;
                }
                case LIKE: {
                    Pattern pattern = Pattern.compile("^.*" + object.getString(VALUE) + ".*$", Pattern.CASE_INSENSITIVE);
                    criteria = Criteria.where(object.getString(KEY)).regex(pattern);
                    arrayList.add(criteria);
                    break;
                }
                case NUM_IS: {
                    criteria = Criteria.where(object.getString(KEY)).is(object.getInteger(VALUE));
                    arrayList.add(criteria);
                    break;
                }
                default: {
                    criteria = Criteria.where("1").is("1");
                    arrayList.add(criteria);
                    break;
                }
            }
        });
        Criteria[] criteria = new Criteria[arrayList.size()];
        arrayList.toArray(criteria);
        return criteria;
    }
 
    /**
     * 生成query
     *
     * @param object object
     * @param flag   条件：true 并且，false 或者
     * @return
     */
    private static Query getQuery(JSONObject object, boolean flag) {
        Query query = new Query();
        // 转为标准json格式
        // 是否有条件集合的键
        boolean b = object.containsKey("conditionList");
       
        if (b) {
            // 存在则获取
            // 条件集合
        List<JSONObject> conditionList = object.getJSONArray("conditionList").toJavaList(JSONObject.class);
            // 是否有值
            if (conditionList.size() > 0) {
                // 生成 Criteria 数组
                Criteria[] criteria = genCriteria(conditionList);
                // 条件 & or |
                if (flag) {
                    query.addCriteria(new Criteria().andOperator(criteria));
                } else {
                    query.addCriteria(new Criteria().orOperator(criteria));
                }
            }
        }
        // 是否存在排序字段
        boolean bsf = object.containsKey("sortField");
        // 是否存在排序类型
        boolean bst = object.containsKey("sortType");
        String sortField = "";
        String sortType = "";
        if (bsf && bst) {
            // 存在则获取
            sortField = object.getString("sortField");
            sortType = object.getString("sortType");
            // 字段判空
            if (StringUtils.isNotBlank(sortField) && StringUtils.isNotBlank(sortType)) {
                // 正序/倒序
                if (ASC.equals(sortType)) {
                    query.with(Sort.by(Sort.Order.asc(sortField)));
                } else {
                    query.with(Sort.by(Sort.Order.desc(sortField)));
                }
            }
        }
        return query;
    }
 
    /**
     * 条件并且
     * 最终在外部调用此方法
     *
     * @param object
     * @return
     */
    public static Query genQueryAnd(JSONObject object) {
        return getQuery(object, true);
    }
 
    /**
     * 条件OR
     * 最终在外部调用此方法
     * 或者查询：相当于MySQL的OR查询条件
     *
     * @param object
     * @return
     */
    public static Query genQueryOr(JSONObject object) {
        return getQuery(object, false);
    }
}
```

