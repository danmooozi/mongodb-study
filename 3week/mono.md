## mongo 전체 컬렉션 문제

**`db.products.countDocuments()`**

[`count()`](https://www.mongodb.com/docs/manual/reference/method/db.collection.count/#mongodb-method-db.collection.count) is equivalent to the `db.collection.find(query).count()` construct.
이런 내용이 있는걸 보아서 find.count() 도 맞다. 그런데 나는 find 하고 count 하면 cursor 크기가 제한이 있어서 전체내용이 안나오럭라고 생각했다.

## explain 결과 문제

```tsx
{
  "executionStats": {
    "nReturned": 1000,
    "executionTimeMillis": 35,
    "totalKeysExamined": 5000,
    "totalDocsExamined": 15000,
    "executionStages": {
      "stage": "FETCH",
      "inputStage": {
        "stage": "IXSCAN",
        "keyPattern": {
          "name": 1
        },
        "indexName": "name_1"
      }
    }
  }
}
```

영어 이슈ㅠ

## 인덱스 쿼리 순서 문제

```tsx
db.gamers.find(
    { "level": { "$gte" : 70 }, "map": "Inferno" } 
).sort( { "points": 1, "group": 1 } )
```

여기에 최적을 구할때는 지난번 시험때 했던 ESR ( equality, Sort, Range ) 를 기반으로 인덱스 순서를 만들어야 한다고 한다.

## insertMany 결과 문제

### **WriteResult**

When passed a single document, [`insert()`](https://www.mongodb.com/docs/manual/reference/method/db.collection.insert/#mongodb-method-db.collection.insert) returns a `WriteResult` object.

### **Successful Results**

The [`insert()`](https://www.mongodb.com/docs/manual/reference/method/db.collection.insert/#mongodb-method-db.collection.insert) returns a [`WriteResult()`](https://www.mongodb.com/docs/manual/reference/method/WriteResult/#mongodb-method-WriteResult) object that contains the status of the operation. Upon success, the [`WriteResult()`](https://www.mongodb.com/docs/manual/reference/method/WriteResult/#mongodb-method-WriteResult) object contains information on the number of documents inserted:

`WriteResult({ "nInserted" : 1 })`

읽어 보면 WriteResult 라는 객체를 뱉고 nInserted 라는 것을 통해서 결과 수를 확인할 수 있다고 한다.

## Attribute patten

### Attribute Pattern

- 상황
    - 비슷한 필드가 많을 때
    - 여러 필드에 걸쳐 조회하고 싶을때
    - 필드가 일부 도큐먼트에만 있을 때
- 해결책
    - 필드를 k/v 짝으로 나누어 서브도큐먼트에 저장
- 예

적용 전

```tsx
{
	"title": "Star Wars",
	"release_US": "1977-05-20T01:00:00+01:00",
	"release_France": "1977-10-19T01:00:00+01:00",
	"release_Italy": "1977-10-20T01:00:00+01:00",
	"release_UK": "1977-12-27T01:00:00+01:00",
}
```

적용 후

```tsx
{
	"title": "Star Wars",
	"director": "George Lucas",
	"releases":
	[
		{
			"location": "USA",
			"date": "1977-05-20T01:00:00+01:00"
		},
		{
			"location": "France",
			"date": "1977-10-19T01:00:00+01:00"
		},
		{
			"location": "Italy",
			"date": "1977-10-20T01:00:00+01:00"
		},
		{
			"location": "UK",
			"date": "1977-12-27T01:00:00+01:00"
		},
	]
}
```

- 장점
    - 인덱싱하기 쉬움
    - 더 적은 인덱스가 필요함
    - 쿼리가 간단해지고 일반적으로 빨라짐

**일부 필드는 유사한 특성을 공유하며 해당 필드 간에 검색을 하고자 합니다. -> 맞습니다. 속성 패턴은 여러 필드가 유사한 특성을 공유하는 시나리오에 가장 적합하며 해당 필드 간에 검색 또는 작업을 수행하고자 합니다. 속성 패턴을 적용하면 관련 속성을 배열 또는 하위 문서로 그룹화하여 보다 효율적이고 논리적으로 데이터를 조회하거나 조작할 수 있습니다.**

https://marsettler.com/mongodb/mongodb-study-week-5/

남이 정리해 둔건데 전체적으로 읽으면 좋을뜻

## Aggregation Framework

하나의 스테이지에서 RAM 100MB 제안이다..

## Sort 할때 null 은 먼저 나온다.

null 이라서 마지막에 나올 줄 알았다.

## 모델링 관련 문제

**MongoDB 환경에서는 제품에 대한 고객 리뷰를 전자 상거래 애플리케이션에 저장하는 작업이 제공됩니다. 각 제품에는 여러 리뷰가 있을 수 있으며 각 리뷰에는 리뷰어의 사용자 이름, 리뷰 내용, 날짜 및 등급(별 1~5개)이 있습니다. 이 경우 가장 적합한 데이터 모델링 접근 방식은 다음 중 무엇입니까?**

**제품 문서의 "리뷰" 필드에 각 리뷰를 별도의 문서로 저장하고 제품당 100개의 리뷰를 제한합니다. -> 정답입니다. 이 방법은 데이터 모델링의 유연성과 문서가 MongoDB의 크기 제한을 초과하지 않도록 하는 것 사이의 균형을 잘 유지합니다. 또한 제품을 조회할 때 리뷰에 대한 효율적인 접근을 제공합니다.**

이거는 어떤식으로 접근해야할지 사실감이 안와서 같이 이야기 하고 싶은 부분이다.

## **findAndModify 문제**

**{new:true} 옵션과 함께 findAndModify 메서드를 사용합니다. -> 정답입니다. {new:true} 옵션은 수정된 문서가 수정될 때 반환되도록 보장합니다. 이는 각 요청이 가장 최근 버전의 문서와 함께 작업할 수 있도록 하기 때문에 동시 요청을 처리하는 데 필수적입니다.**

아니 동시성 처리하는데 new 옵션이 필요한지 어떻게 아냐고.

## **mongorestore**

**mongorestore -> 맞습니다. 이 명령은 mongodump 또는 유사한 도구에 의해 생성된 이진 백업에서 데이터를 복원하는 데 사용되며, BSON 파일에 저장된 컬렉션을 MongoDB 클러스터에 직접 추가할 수 있습니다.**

```tsx
See also:
mongodump, which provides the corresponding binary data export capability.
```

## **sortByCount**

**$sortByCount는 특정 필드(이 경우 도시)별로 문서를 그룹화하고 각 그룹의 문서 수를 세어 내림차순으로 결과를 카운트별로 정렬하는 간단한 방법입니다.**
