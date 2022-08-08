---
title: Meilisearch Doc
date: 2022-08-08 23:07:16
tags: Meilisearch
---

Meili 是在挪威神話中的神，指"可愛的人"，是托爾的兄弟。
---

----

### Preview

搜尋速度快、全文檢索、支援中文、容易安裝和維護

![MeiliSearchPreview.gif](https://raw.githubusercontent.com/SaxonLu/SaxonLu.github.io/main/data/img/64EA8A61-87F1-5C4A-B68A-57C0F52DD753.gif)



## Here We Go !


### 安裝Server的方式

官方[Server Install Doc](https://docs.meilisearch.com/learn/getting_started/quick_start.html#setup-and-installation)有提供多種方式

我這邊使用的是Docker的方式

``` docker
#取得Docker Image
docker pull getmeili/meilisearch:v0.28

#建立Container
docker run -p 7700:7700 -d MEILI_MASTER_KEY='MASTER_KEY' getmeili/meilisearch
```

這邊需要提到的是 `MEILI_MASTER_KEY` 是用來設定Server的金鑰，
如果於建立Container時添加這個屬性，
未來對這個Server操作的行為都需要帶上這個金鑰。

以直接對MeiliSearch Server API Request 的情況來說就需要帶上`X-Meili-API-Key`

![PostManRequestWithKeySample.png](https://raw.githubusercontent.com/SaxonLu/SaxonLu.github.io/main/data/img/imgimage-20210722152523831.png)

---

### 匯入搜尋資料

這邊提供了兩種方式

一個是藉由JSON, NDJSON, 或 CSV 格式的檔案匯入
另一種則是藉由SDK或API進行建立資料

實際操作如下

這邊使用Golang作為示範

官方[Add Documents Doc](https://docs.meilisearch.com/learn/getting_started/quick_start.html#add-documents)有提供其他語言的範例

``` shell
go get -u github.com/meilisearch/meilisearch-go
```

### 批量檔案匯入

``` go
package main

import (
  "os"
  "encoding/json"
  "io/ioutil"

  "github.com/meilisearch/meilisearch-go"
)

func main() {
  client := meilisearch.NewClient(meilisearch.ClientConfig{
    Host: "http://127.0.0.1:7700",
  })

  jsonFile, _ := os.Open("movies.json")
  defer jsonFile.Close()

  byteValue, _ := ioutil.ReadAll(jsonFile)
  var movies []map[string]interface{}
  json.Unmarshal(byteValue, &movies)

  _, err := client.Index("movies").AddDocuments(movies)
  if err != nil {
      panic(err)
  }
}
```
### 藉由SDK寫入

```go
package main

import (
	"fmt"
	"os"

	"github.com/meilisearch/meilisearch-go"
)

func main() {
	client := meilisearch.NewClient(meilisearch.ClientConfig{
		Host: "http://127.0.0.1:7700",
	})

	index := client.Index("game")

	documents := []map[string]interface{}{
		{"id": 1, "title": "星海爭霸", "genres": []string{"即時戰略", "科幻", "戰爭"}},
		{"id": 2, "title": "仙境傳說RO", "genres": []string{"角色扮演", "MMORPG", "線上遊戲"}},
		{"id": 3, "title": "暗黑破壞神", "genres": []string{"刷裝", "角色扮演", "動作"}},
		{"id": 4, "title": "英雄聯盟", "genres": []string{"MOBA", "DOTA"}},
		{"id": 5, "title": "魔獸世界", "genres": []string{"角色扮演", "MMORPG", "科幻", "線上遊戲"}},
		{"id": 6, "title": "絕對武力", "genres": []string{"第一人稱射擊遊戲", "FPS"}},
	}
	task, err := index.AddDocuments(documents)
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	fmt.Println(task.TaskUID)
}
```

其中欄位並非**固定**除了用來識別的**ID**外

``` go 
package main

import (
	"fmt"
	"os"

	"github.com/meilisearch/meilisearch-go"
)

func main() {
	client := meilisearch.NewClient(meilisearch.ClientConfig{
		Host: "http://127.0.0.1:7700",
	})

	index := client.Index("freeDoc2")

	documents := []map[string]interface{}{
		{"id": 1, "忘了填": "沒關係"},
		{"id": 2, "title": "棉豆腐", "好評推薦": []string{"你", "我", "還有隻貪睡的貓"}},
		{"id": 3, "馬斯克": "我到底該不該收購twitter還是在操作一波讓他降價呢?"},
	}
	task, err := index.AddDocuments(documents)
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	fmt.Println(task.TaskUID)
}

```

![FreeDoc2Sample.png](https://raw.githubusercontent.com/SaxonLu/SaxonLu.github.io/main/data/img/freedoc.JPG)

---

### 編輯資料

```go
	documents := []map[string]interface{}{
		{
			"id":     6,
			"title":  "絕對武力Counter-Strike",
			"genres": []string{"第一人稱射擊遊戲", "FPS", "comedy⚡️⚡️"},
		},
	}
	client.Index("game").UpdateDocuments(documents, "")

```

![updatedoc.png](https://raw.githubusercontent.com/SaxonLu/SaxonLu.github.io/main/data/img/updateDoc.JPG)

---

### 搜尋

這邊先提一下目前架設的環境

**流程圖**

![mermaid.png]()

所以之後的操作會以SDK對Meilisearch Server行為為主

搜尋的重點
1.對哪個集合操作 **(Index)**
2.搜尋的項目方式 **(search parameters)**

``` go
type SearchRequest struct {
	Offset                int64
	Limit                 int64
	AttributesToRetrieve  []string //對index資料建立時的類別做選擇(例如movie的OVERVIEW ,POSTER 等等)
	AttributesToCrop      []string //Crop的部分是在針對該關鍵字的前後語句長度
	CropLength            int64
	CropMarker            string
	AttributesToHighlight []string //對指定的類別做搜尋結果做重點提示
	HighlightPreTag       string //對其搜尋結果做頭,
	HighlightPostTag      string //和尾的重點提示HTML結構 盡量一起使用且須注意前後是否相同
	Filter                interface{}
	ShowMatchesPosition   bool
	Facets                []string //like count & group by 
	PlaceholderSearch     bool
	Sort                  []string
}

// SearchResponse is the response body for search method
type SearchResponse struct {
	Hits               []interface{} `json:"hits"` //被搜尋到的結果
	EstimatedTotalHits int64         `json:"estimatedTotalHits"`
	Offset             int64         `json:"offset"`
	Limit              int64         `json:"limit"`
	ProcessingTimeMs   int64         `json:"processingTimeMs"`
	Query              string        `json:"query"`
	FacetDistribution  interface{}   `json:"facetDistribution,omitempty"`
}
```

**Filter**的操作就比較多樣化了

``` go 
resp, err := client.Index("movies").Search("thriller", &meilisearch.SearchRequest{
  Filter: [][]string{
    []string{"genres = Horror", "genres = Mystery"},
    []string{"director = \"Jordan Peele\""},
  },
})
```

**Search Sample**

Basic Search Request
``` go 
func main() {
    searchRes, err := client.Index("movies").Search("wonder",
        &meilisearch.SearchRequest{
            AttributesToHighlight: []string{"*"},
        })
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }

    fmt.Println(searchRes.Hits)
}
```
Response
```JSON
{
    "hits": [
        {
            "id": 2,
            "title": "Wonder Woman",
            "genres": ["Action", "Adventure"],
            "_formatted": {
                "id": 2,
                "title": "<em>Wonder</em> Woman"
            }
        }
    ],
    "offset": 0,
    "limit": 20,
    "processingTimeMs": 0,
    "query": "wonder"
}
```


Filter Search Request
```go
searchRes, err := index.Search("wonder",
    &meilisearch.SearchRequest{
        Filter: "id > 1 AND genres = Action",
    })
```
Response
```json
{
  "hits": [
    {
      "id": 2,
      "title": "Wonder Woman",
      "genres": ["Action","Adventure"]
    }
  ],
  "offset": 0,
  "limit": 20,
  "estimatedTotalHits": 1,
  "processingTimeMs": 0,
  "query": "wonder"
}
```
自己測試的結果
中文檢索的時候有時會無法搜尋到Array內中間字，
且對其字數有些例外狀況
可能是中文轉型的限制

**Sample**

前二字
![top2](https://raw.githubusercontent.com/SaxonLu/SaxonLu.github.io/main/data/img/top2.JPG)

前三字
![top3](https://raw.githubusercontent.com/SaxonLu/SaxonLu.github.io/main/data/img/top3.JPG)

中間二字
![mid2](https://raw.githubusercontent.com/SaxonLu/SaxonLu.github.io/main/data/img/mid2.JPG)

尾二字
![end2](https://raw.githubusercontent.com/SaxonLu/SaxonLu.github.io/main/data/img/end2.JPG)

---

### 設置

**停用詞**

```go
client.Index("movie").UpdateSettings(&meilisearch.Settings{
    StopWords:[]string{"the","and","的"}
})    
```

**排序規則**
```go
client.Index("movie"). UpdateRankingRules([]string{
  "words",
  "typo",
  "proximity",
  "attribute",
  "sort",
  "exactness",
  "release_date:desc"
})   
```

這些設置可以有效的提高搜尋效果，比如使用停用詞之前，搜尋**開源的書籍**命中不了**開源書籍**，加了停用詞即可命中，因為配對時忽略了輸入内容包含的停用詞(無用詞 *的*）。

另外，功能上沒有建議/關聯字（suggest），可以通過新建 index+searchableAttributes達到。

---

### 部屬

官方文檔提供了多種部屬文件

請參照

[GCP](https://docs.meilisearch.com/learn/cookbooks/gcp.html)

---

### 結語

ES 做為老牌搜索引擎，功能基本滿足，但複雜，重量級，適合大數據量且上手慢。
Meili 設計目標針對數據在 500GB 左右的搜尋需求，極快，單文件，超輕量。
