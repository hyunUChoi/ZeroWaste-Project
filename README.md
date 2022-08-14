# 🌱 ECO:BACK


## ✅ 서비스 소개
### ✔ 서비스명 
### 지도 라이브러리 기반 제로웨이스트 실천 안내 서비스
<br>

## ✅ 주요 기능
* 지도 상에 제로웨이스트 매장 위치, 정보를 음식 종류별 또는 검색 기능을 통해 사용자가 매장을 찾을 수 있도록 하였습니다.
* 영수증 인증과 매장 리뷰를 등록시킬 수 있는 사용자 커뮤니티 게시판을 구현하였습니다.
* 제로웨이스트 실천 횟수에 따라 사용자의 에코등급 시각화할 수 있도록 구현하였습니다.
* 에코 등급에 따른 제로웨이스트 생필품 샵에서 사용할 수 있는 쿠폰을 발급 받을 수 있도록 구현하였습니다.
* 신규 매장등록 시 매장의 위치를 확인할 수 있도록 하였으며 위치를 지도 상에 자동 매핑되로록 구현하였습니다.
* 사용자의 편의성을 위해 웹과 앱화면 모두 구동 가능하도록 반응형 웹사이트로 구현하였습니다.
<br>

## ✅ 담당 역할
![Langauge:Java](https://img.shields.io/badge/Langauge-Java-green) ![DB:Oracle](https://img.shields.io/badge/DB-Oracle-yellow) ![API:Kakao MAP API](https://img.shields.io/badge/API-Kakao%20MAP%20API-orange) ![Server:Apache Tomcat](https://img.shields.io/badge/Server-Apache%20Tomcat-blue)
* Oracle을 활용한 테이블 구조 설계
* SQL Query문을 통한 데이터 입출력 구현
* Mybatis를 이용한 MVC 패턴 분리 및 웹 서비스 흐름 설계
* JSP를 이용한 에코등급 시각화, 쿠폰 발급/신규 매장 등록 페이지 구현
* Kakao MAP API를 이용한 지도 출력 및 입력 주소에 따른 위/경도 값 자동 매핑 기능 구현
<br><br>

## ✅ 트러블슈팅
* 데이터베이스 설계<br>
	- 처음으로 데이터의 특성에 맞게 여러개 테이블을 설계하고 각 컬럼별 제약조건과 테이블 간 Foreign key 연결 작업을 수행하다보니 후에 잘못된 구조라는 것을 알아도 쉽사리 바꾸기가 어려웠다.
	- 초반에 DB 구조를 탄탄하게 구성하지 않았기에 여러번 DB 구조를 수정하고 DROP하는 일이 생겼다.
	- 이를 통해 설계 초반부터 어떠한 데이터를 가지고 올 것인지, 어떤 테이블에 어떠한 데이터를 넣어야 흐름이 자연스럽게 넘어오는지 등 데이터의 흐름을 파악하는 것이 중요하다라는 것을 배웠다.
<br>

* 매장 리뷰 등록 후 포인트 적립 미반영<br>
    - 커뮤니티 게시판에 이용한 매장의 영수증 인증 시 데이터베이스에는 포인트 증가 및 등급 상향 반영에는 성공하였지만 웹 페이지 상에는 처음 로그인한 상태 그대로의 포인트와 에코 등급이 출력되고 재로그인을 해야지만 반영되는 문제점이 발생하였다.
    - 이 문제를 해결하기 위해 기존 로그인 시 생성하였던 Session을 이용자가 커뮤니티 게시판에 인증 성공함과 동시에 업데이트 시키는 방법을 통해 해결하였다.
 ```jsx
 // Session Update
 UserVO user = (UserVO)session.getAttribute("login");
	
        // 방문 매장명, 영수증 및 음식 사진, 리뷰 등록
		cvo.setId(user.getId());
		cvo.setStoreName(storeName);
		cvo.setFileName(fileName);
		cvo.setReview(review);
		
        // 인증글 DB Insert SQL
		int cnt = cdao.write(cvo);
		
		if(cnt > 0) {
            // 게시판 인증 성공 시 포인트 증가 SQL 실행 후 에코 등급 시각화 페이지로 이동
			udao.pointup(user.getId());
			response.sendRedirect("GoTree");
		} else {
            // 인증 실패 시 게시판 글쓰기 페이지로 복귀
			response.sendRedirect("GoWrite");
		}
 ```
 <br>
 
* 신규 매장 등록 시 위/경도 값 받아오기<br>
    - 지도 상 매장의 위치를 나타내기 위해서는 해당 매장의 위/경도 값이 필요했지만 신규 매장 등록을 원하는 이용자에게 직접 본인 매장의 위/경도 값을 기재해 달라는 것은 이용자 편의성에 위배되는 것으로 판단되었다.
    - 이 문제를 해결하기 위해 Kakao MAP API Geocoder 라이브러리를 통해 이용자가 본인 매장의 주소를 입력하면 해당 위치를 이용자가 확인한 후 주소가 다르다면 Marker를 이동하여 정확한 위/경도 값을 받아올 수 있도록 구현하였다.
 ```jsx
 <script>
    var mapContainer = document.getElementById('map'), // 지도를 표시할 div 
	mapOption = {
			center : new kakao.maps.LatLng(35.15119034201585, 126.92515360052161), // 지도의 중심좌표
			level : 3// 지도의 확대 레벨
			};

			// 지도를 생성합니다    
			var map = new kakao.maps.Map(mapContainer, mapOption);

			// 주소-좌표 변환 객체를 생성합니다
			var geocoder = new kakao.maps.services.Geocoder();
			var btn = document.getElementById('btn');
			var address = '';

		    btn.addEventListener('click',function() { address = document.getElementById('addr').value;
				// 주소로 좌표를 검색합니다
				geocoder.addressSearch(address,function(result, status) {
				// 정상적으로 검색이 완료됐으면 
				if (status === kakao.maps.services.Status.OK) {
					var coords = new kakao.maps.LatLng(result[0].y, result[0].x);
					// 결과값으로 받은 위치를 마커로 표시합니다
					var marker = new kakao.maps.Marker({
									map : map,
									position : coords
							});

            // 인포윈도우로 장소에 대한 설명을 표시합니다
			var infowindow = new kakao.maps.InfoWindow({
						content : '<div style="width:150px;text-align:center;padding:6px 0;">우리가게</div>'
											           });
			infowindow.open(map, marker);

			// 지도의 중심을 결과값으로 받은 위치로 이동시킵니다
			map.setCenter(coords);
										
			// 위도 & 경도 찾기
			var lat = document.getElementById('lat')
			var lng = document.getElementById('lng')
			lat.value = result[0].y
			lng.value = result[0].x
					}
				});
			});
    </script>
 ```

<details>
<summary><h2>🧾More Details</h2></summary>

## ✅ 프로젝트 기간
2022.06.03 ~ 2022.06.18
<br><br>
    
## ✅ 화면 구성

### 회원가입 / 메인화면 / 사용자 튜토리얼 화면
![image](https://user-images.githubusercontent.com/103620466/182588812-326be119-90cb-4264-b3f1-bb7eb059888f.png)
<br><br>

### 매장 보기 화면 (전체 매장 / 카테고리 선택 / 매장명 검색)
![image](https://user-images.githubusercontent.com/103620466/182589092-43fdf433-026b-47da-9d48-a5c5105ecdf3.png)
<br><br>

### 커뮤니티 게시판 화면 / 리뷰 등록 / 리뷰 수정
![image](https://user-images.githubusercontent.com/103620466/182589351-00081d31-ca43-4193-9fb2-23fa1b506990.png)
<br><br>

### 등급 시각화 / 리워드 화면 / 신규 매장등록 화면
![image](https://user-images.githubusercontent.com/103620466/182589764-d97e7c59-957b-47aa-a884-1e62ba9cd57d.png)
<br><br>
</details>
