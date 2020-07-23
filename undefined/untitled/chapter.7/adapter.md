# 어댑터\(Adapter\) 패턴

게시판 통합 검색기능을 DB를 이용해 구현했다.

![](../../../.gitbook/assets/image%20%2849%29.png)

검색 속도 성능에 문제가 발생해서 Tolr 라는 오픈 소스 검색 서버를 도입하기로 했다.

![](../../../.gitbook/assets/image%20%2843%29.png)

클라이언트가 요구하는 인터페이스\(SerachService\)와 재사용하려는 모듈\(TolrClient\)가 일치하지 않을 때 사용할 수 있는 패턴이 어댑터 패턴이다.

![](../../../.gitbook/assets/image%20%2847%29.png)

어댑터에 해당하는 SearchServiceTolrAdapter 클래스는 TolrClient를 SearchService 인터페이스에 맞춰 주는 책임을 갖는다.

```java
public class SearchSErviceTolrAdapter implements SearchService {
    private TolrClient tolrClient = new TolrClient();
    
    public SearchResult search(String keyword) {
        // keyword를 tolrClient가 요구하는 형식으로 변환
        TolrQuery tolrQuery = new TolrQuery(keyword);
        
        //TolrClient 기능 실행
        QueryResponse response = tolrClient.query(tolrQuery);

        // TolrClient의 결과를 SearchResult로 변환    
        SearchResult result = convertToResult(response);
        return result;
    }
    
    private SearchResult convertToResult(QueryResponse response) {
        List<TolrDocument> tolrDocs = response.getDocumentList().getDocuments();
        List<SearchDocument> docs = new ArrayList<SearchDocument>();
        for (TolrDocument tolrDoc: tolrDocs) {
            docs.add(new SearchDocument(tolrDoc.getId(), ...);
        }
        return new SearchResult(docs);
    }
}
```

어댑터 패턴은 개방 폐쇄 원칙을 따를 수 있도록 도와준다.

