---
title: sensitive_words
date: 2016-03-17 11:31:15
tags: [algorithm,python,DFA]
categories: [algorithm,python]
---

## 敏感词

### 敏感词检测算法

思路想法来源于[这里](http://blog.csdn.net/chenssy/article/details/26961957)

## Python代码实现

### 建立前缀树
```python
def _build_prefix_tree(sensitive_word_list):
    '''
    Parameter: sensitive_word_list(list)
    Return:    embedded dictionary , stands for prefix tree
    '''
    if sensitive_word_list:
        prefix_tree = {}
        for str_word in sensitive_word_list:
            each_word = unicode(str_word)
            word_len = len(each_word)
            now_tree = prefix_tree
            for i in xrange(word_len):
                if i != (word_len - 1):
                    letter_dict = now_tree.get(each_word[i])
                    if letter_dict == None:
                        letter_dict = {}
                        letter_dict["isEnd"] = 0
                        now_tree[each_word[i]] = letter_dict
                    now_tree = now_tree.get(each_word[i])
                else:
                    letter_dict = now_tree.get(each_word[i])
                    if letter_dict == None:
                        letter_dict = {}
                        letter_dict["isEnd"] = 1
                        now_tree[each_word[i]] = letter_dict
                    else:
                        letter_dict["isEnd"] = 1
                    now_tree = now_tree.get(each_word[i])
        return prefix_tree
    else:
        return {}
```

### 判断输入是否包含敏感词
```python
def _is_sensitive(prefix_tree, input_word):
    '''
    internal call
    '''
    if input_word:
        one_word = unicode(input_word)
        for i in xrange(len(one_word)):
            found_dict = prefix_tree.get(one_word[i])
            if found_dict == None:
                return False
            else:
                prefix_tree = found_dict
                is_end = prefix_tree.get("isEnd")
                if is_end == 1:
                    return True
        return False
    else:
        return False
```

### 测试代码

```python
if __name__ == "__main__":
    sensitive_word_list = ["草他妹","中国男足","奶奶的"]
    mlogger = Logging("testService", "console").logger
    prefix_tree = _build_prefix_tree(sensitive_word_list,mlogger)
    print _is_sensitive(prefix_tree,"中女")
    print _is_sensitive(prefix_tree,"中男")
    print _is_sensitive(prefix_tree,"中国男足")
    print _is_sensitive(prefix_tree,"fuck")
    print _is_sensitive(prefix_tree,"fuc")
    print _is_sensitive(prefix_tree,"中国男足实在是臭的不得了，奶奶的草他妹啊，怎么不去死啊，真不要脸")
```

## 敏感词动态更新检查策略

### BaseHandler

帖子写入数据库前检查，如果没有初始化，调用_update_sensitive。
如果server实例本地内存保存的敏感词版本太老了，则调用_update_sensitive更新
```python
@gen.coroutine
def _update_sensitive(self):
    self.application.logger.info("_update_sensitive entered")
    version,words_list = yield self.servers["sensitive"].get_sensitive_words_list(self.auth_header)

    if self.application.sensitive_words_version != version:
        self.application.sensitive_words_tree = _build_prefix_tree(words_list)
        self.application.sensitive_words_version = version
        self.application.sensitive_words_last_update = int(time.time() * 1000)
    else:
        self.application.logger.info("sensitive words version {0} not changed"\
                                     .format(str(self.application.sensitive_words_version)))

@gen.coroutine
def detect_sensitive(self,content):
    if content == None:
        raise gen.Return(False)

    #毫秒
    current_time = int(time.time() * 1000)
    self.application.logger.info("current time {0}".format(current_time))
    self.application.logger.info("record_last_update_time {0}".format(self.application.sensitive_words_last_update))

    if self.application.sensitive_words_last_update == None:
        yield self._update_sensitive()
    elif (current_time - self.application.sensitive_words_last_update > 12 * 3600 * 1000):
        yield self._update_sensitive()

    if self.application.sensitive_words_tree:
        ret = _is_sensitive(self.application.sensitive_words_tree,content)
        raise gen.Return(ret)
    else:
        raise gen.Return(False)
```

### 帖子写入计算

帖子写入数据库前，计算是否有敏感词，以及敏感词版本

```python
# 1.1过滤敏感字
is_sensitive = yield self.detect_sensitive(param.get("textContent"))
if is_sensitive:
    param["is_sensitive"] = SENSITIVE
    param["sensitive_version"] = self.sensitive_words_version
else:
    param["is_sensitive"] = NOT_SENSITIVE
    param["sensitive_version"] = self.sensitive_words_version
```

### 帖子读取计算

1. 读取敏感词版本，是否敏感
2. 如果版本和本地保存版本一致，直接操作
3. 如果版本和本地保存版本不一致，重新计算结果，更新数据库，按照新结果操作

```python
#敏感词检查
is_sensitive = p.get("is_sensitive")
version = p.get("sensitive_version")
self.logger.debug("{0} is sensitive {1}".format(post_id,str(is_sensitive)))

#毫秒
current_time = int(time.time() * 1000)
self.application.logger.info("current time {0}".format(current_time))
self.application.logger.info("record_last_update_time {0}".format(self.sensitive_words_last_update))

if self.sensitive_words_last_update == None:
    yield self._update_sensitive()
elif (current_time - self.sensitive_words_last_update > 12 * 3600 * 1000):
    yield self._update_sensitive()

if version != self.sensitive_words_version:
    self.logger.debug("old sensitive version {0}, new version {1}"\
                          .format(version,self.sensitive_words_version))
    if self.sensitive_words_tree:
        new_analyze_result = _is_sensitive(self.sensitive_words_tree,p.get("textContent"))
        if new_analyze_result == True:
            yield self.posts.update({"_id": ObjectId(post_id)},\
                                    {"$set":{"sensitive_version": self.sensitive_words_version,\
                                             "is_sensitive":SENSITIVE}})
            self.response(SENSITIVE_POST_NOT_ALLOWED_TO_SHOW)
        else:
            yield self.posts.update({"_id": ObjectId(post_id)},\
                                    {"$set":{"sensitive_version":self.sensitive_words_version, \
                                             "is_sensitive":NOT_SENSITIVE}})
else:
    if is_sensitive == SENSITIVE:
        self.response(SENSITIVE_POST_NOT_ALLOWED_TO_SHOW)
    else:
        pass
```
<div class="post-spread">
<div class="ds-share flat" data-thread-key="2016/03/12/vscode-c/" data-title="Mac下VSCode配置为C++ IDE" data-content="" data-url="http://babayetu.github.io/2016/03/12/vscode-c/">
  <div class="ds-share-inline">
    <ul  class="ds-share-icons-16"><li data-toggle="ds-share-icons-more"><a class="ds-more" href="javascript:void(0);">分享到：</a></li><li><a class="ds-weibo" href="javascript:void(0);" data-service="weibo">微博</a></li><li><a class="ds-qzone" href="javascript:void(0);" data-service="qzone">QQ空间</a></li><li><a class="ds-qqt" href="javascript:void(0);" data-service="qqt">腾讯微博</a></li><li><a class="ds-wechat" href="javascript:void(0);" data-service="wechat">微信</a></li>
    </ul>
    <div class="ds-share-icons-more"></div>
  </div>
</div>
</div>