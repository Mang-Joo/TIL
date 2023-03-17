# HTTP 엔터티

### 미디어타입

HTTP 요청과 응답 메시지는 전송하는 데이터의 유형을 나타내는 MIME(Multipurpose Internet Mail Extensions) 타입을 포함하고 있습니다. MIME 타입은 미디어타입/서브타입 형태로 작성되며, 대표적인 미디어타입으로는 다음과 같은 것들이 있습니다.

* text/plain : 일반 텍스트
* text/html : HTML 문서
* application/json : JSON 데이터
* image/jpeg : JPEG 이미지
* audio/mp3 : MP3 오디오
* video/mp4 : MP4 비디오 등

HTTP 클라이언트는 서버로부터 받은 MIME 타입 정보를 이용하여 데이터를 올바르게 처리할 수 있습니다. MIME 타입은 브라우저가 적절한 액션을 취할 수 있도록 하여, 예를 들어 새 창이나 다운로드 등의 동작을 수행합니다.

### Charset

Charset은 문자 데이터가 인코딩되는 방식을 나타냅니다. HTTP 요청과 응답 메시지에는 Content-Type 헤더 필드가 있으며, 이 필드에는 미디어타입 정보와 함께 Charset 정보도 포함될 수 있습니다. 대표적인 Charset으로는 UTF-8, EUC-KR, ISO-8859-1 등이 있으며, Charset은 문서의 표현 방법에 대한 정보를 제공합니다.

올바른 Charset 정보를 포함하는 것은 문서의 표현 방법을 지정하는 것으로, 이를 통해 브라우저는 문서를 올바르게 해석할 수 있게 됩니다. 또한, Charset 정보가 없는 경우, 브라우저는 기본 Charset으로 UTF-8을 사용하게 됩니다.

### Content-Length

Content-Length는 HTTP 메시지 본문의 크기를 바이트 단위로 나타냅니다. HTTP 요청의 경우, Content-Length는 요청 메시지 본문의 크기를 나타내며, HTTP 응답의 경우, Content-Length는 응답 본문의 크기를 나타냅니다.

올바른 Content-Length 정보를 포함하는 것은 데이터 전송의 안정성과 효율성을 보장하는데 중요한 역할을 합니다. 이 정보가 없으면, 클라이언트와 서버 간에 데이터 전송이 불완전하게 되거나, 메모리 누수 등의 문제가 발생할 수 있습니다. 따라서, Content-Length 정보를 정확하게 포함하여 데이터 전송의 안정성을 확보하는 것이 중요합니다.
