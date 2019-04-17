---
layout: post
title: Kakao Geocode API, 주소로 위도 경도 java로 파싱하기 
comments: true
categories : [Java]
tags: [java]

---

Kakao Geocode API로 부터 받은 데이터를 직렬화 하고 JSON객체로 바꾸고 특정 데이터를 얻어오는 작업을 할 것입니다.  
Kakao API를 이용하기 위해선 API Key가 필요한데, [이곳](https://developers.kakao.com) 에서 얻을 수 있습니다.  

카카오 지도 API로부터 주소로 위도 경도를 받아오기 위해선 [이곳](https://developers.kakao.com/docs/restapi/local#주소-검색)에 공식 예제가 있습니다.  

### 받아온 주소 데이터를 JSON으로
헤더에 발급받은 API키를 등록 후 POSTMAN으로 요청을 날려보겠습니다.  
>https://dapi.kakao.com/v2/local/search/address.json?query=판교역로 235

응답받은 데이터는 이러합니다.  

~~~json
{
    "meta": {
        "is_end": true,
        "total_count": 1,
        "pageable_count": 1
    },
    "documents": [
        {
            "road_address": {
                "undergroun_yn": "N",
                "road_name": "판교역로",
                "underground_yn": "N",
                "region_2depth_name": "성남시 분당구",
                "zone_no": "13494",
                "sub_building_no": "",
                "region_3depth_name": "삼평동",
                "main_building_no": "235",
                "address_name": "경기 성남시 분당구 판교역로 235",
                "y": "37.40209529907863",
                "x": "127.10863694633468",
                "region_1depth_name": "경기",
                "building_name": "에이치스퀘어 엔동"
            },
            "address_name": "경기 성남시 분당구 판교역로 235",
            "address": {
                "b_code": "4113510900",
                "region_3depth_h_name": "삼평동",
                "main_address_no": "681",
                "h_code": "4113565500",
                "region_2depth_name": "성남시 분당구",
                "main_adderss_no": "681",
                "sub_address_no": "",
                "region_3depth_name": "삼평동",
                "address_name": "경기 성남시 분당구 삼평동 681",
                "y": "37.40206645815382",
                "x": "127.10864594007738",
                "mountain_yn": "N",
                "zip_code": "463400",
                "region_1depth_name": "경기",
                "sub_adderss_no": ""
            },
            "y": "37.40209529907863",
            "x": "127.10863694633468",
            "address_type": "ROAD_ADDR"
        }
    ]
}
~~~

위의 데이터에서 `Y` 좌표 값과 `X` 좌표값이 필요하기 때문에 해당 데이터만 가져오도록 하겠습니다.

우선 KAKAO 지도 API로 요청을 보내고 응답을 받습니다.
~~~java
    String APIKey = "발급받은 API 키";    
    
    HashMap<String, Object> map = new HashMap<>(); //결과를 담을 map

    try {
        String apiURL = "https://dapi.kakao.com/v2/local/search/address.json?query=" 
                        + URLEncoder.encode(address, "UTF-8");
        
        HttpResponse<JsonNode> response = Unirest.get(apiURL)
                .header("Authorization", APIKey)
                .asJson();
~~~

받아온 데이터를 JSON 객체로 변환하기 위해 `ObjectMapper`를 사용합니다.

~~~java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.configure(DeserializationFeature.ACCEPT_SINGLE_VALUE_AS_ARRAY, true); 
~~~

위의 `objectMapper.configure(DeserializationFeature.ACCEPT_SINGLE_VALUE_AS_ARRAY, true);` 는   
단일 리스트 객체를 싱글 값과 같게 인식합니다.     
ex) `"fruits" : ["apple"] 를 "fruits" : "apple"` 로 인식

### 지정된 VO에 응답받은 데이터를 셋팅

지정된 형식에 잘 셋팅하기 위해
~~~java
KakaoGeoRes bodyJson = objectMapper.readValue(response.getBody().toString(), KakaoGeoRes.class);
~~~
`KakaoGeoRes`에 응답받은 데이터를 잘 셋팅하도록 합니다.

~~~java
@Data
public class KakaoGeoRes {
    private HashMap<String, Object> meta;
    private List<Documents> documents;
}

@Data
class Documents {
    private HashMap<String, Object> address;
    private String address_type;
    private Double x;
    private Double y;
    private String address_name;
    private HashMap<String, Object> road_address;
}
~~~
`X, Y`값만 필요했기 때문에 위와 같이 작성했습니다.  
key가 응답받은 데이터와 다르면 에러가 나기 때문에 사용하지 않는 key 도 선언합니다.

### X,Y 값에 접근
그 후 
~~~java
bodyJson.getDocuments().get(0).getX()
bodyJson.getDocuments().get(0).getY()
~~~
위와같이 접근할 수 있습니다.

읽어주셔서 감사합니다.  
혹 오류나 질문이 있다면 편하게 코멘트 부탁드리겠습니다!🙆‍♂️
