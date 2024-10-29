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
                  def aValues = doc['A.a'];
                  for (def item : aValues) {
                    if (!params.a1.contains(item)) {
                      return false;
                    }
                  }
                  return true;
                  """,
                  "params": {
                    "a1": ["a", "b", "c", "d"]
                  }
                }
              }
            },
            {
              "script": {
                "script": {
                  "source": """
                  def bValues = doc['A.b'];
                  for (def item : bValues) {
                    if (!params.a2.contains(item)) {
                      return false;
                    }
                  }
                  return true;
                  """,
                  "params": {
                    "a2": ["b", "e","f","x"]
                  }
                }
              }
            },
            {
              "script": {
                "script": {
                  "source": """
                  def cValues = doc['A.c'];
                  for (def item : cValues) {
                    if (!params.a3.contains(item)) {
                      return false;
                    }
                  }
                  return true;
                  """,
                  "params": {
                    "a3": ["c", "g", "h"]
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
                  int count = 0;
                  for (def item : params.a1) {
                    if (doc['A.a'].contains(item)) {
                      count++;
                    }
                  }
                  return count == params.a1.size();
                  """,
                  "params": {
                    "a1": ["a", "b", "c", "d"]
                  }
                }
              }
            },
            {
              "script": {
                "script": {
                  "source": """
                  int count = 0;
                  for (def item : params.a2) {
                    if (doc['A.b'].contains(item)) {
                      count++;
                    }
                  }
                  return count == params.a2.size();
                  """,
                  "params": {
                    "a2": ["b", "d", "e"]
                  }
                }
              }
            },
            {
              "script": {
                "script": {
                  "source": """
                  int count = 0;
                  for (def item : params.a3) {
                    if (doc['A.c'].contains(item)) {
                      count++;
                    }
                  }
                  return count == params.a3.size();
                  """,
                  "params": {
                    "a3": ["c", "g", "h"]
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
