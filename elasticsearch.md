DELETE my_index

PUT /my_index
{
  "mappings": {
    "properties": {
      "A": {
        "type": "nested",
        "properties": {
          "a": { "type": "keyword" },
          "b": { "type": "keyword" },
          "c": { "type": "keyword" }
        }
      }
    }
  }
}


POST /my_index/_doc/1
{
  "A": [
    { "a": ["a", "c", "d"], "b": ["b", "e", "f"], "c": ["c", "g", "h"] }
  ]
}


POST /my_index/_doc/2
{
  "A": [
    { "a": ["a", "c", "d","Z"], "b": ["b", "e", "f"], "c": ["c", "g", "h"] }
  ]
}



GET /my_index/_search
{
  "query": {
    "nested": {
      "path": "A",
      "query": {
        "bool": {
          "must": [
            {
              "script": {
                "script": {
                  "source": """
                  def a1 = params.a1;
                  if(a1 != null&&!a1.isEmpty()&&!a1.contains("All")){
                    // 获取 A.a 字段的值
                    def aValues = doc['A.a'];
                    // 如果 aValues 为 null 或为空，直接返回 false
                    if (aValues == null || aValues.isEmpty()) {
                      return false;
                    }
                    // 检查 aValues 中的所有值是否都在 a1 中
                    for (def item : aValues) {
                      if (!a1.contains(item)) {
                        return false;
                      }
                    }
                  }
                  def b1 = params.b1;
                  if(b1 != null&&!a1.isEmpty()&&!b1.contains("All")){
                    // 获取 A.a 字段的值
                    def bValues = doc['A.b'];
                    // 如果 aValues 为 null 或为空，直接返回 false
                    if (bValues == null || bValues.isEmpty()) {
                      return false;
                    }
                    // 检查 aValues 中的所有值是否都在 a1 中
                    for (def item : bValues) {
                      if (!b1.contains(item)) {
                        return false;
                      }
                    }
                  }
                  return true;
                  """,
                  "params": {
                    "a1": ["All"],
                    "b1": ["b", "e", "f"]
                  }
                }
              }
            }
          ]
        }
      }
    }
  }
}
