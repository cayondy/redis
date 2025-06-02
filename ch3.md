# 03. 레디스 기본 개념
  
## 레디스의 자료구조 

- string
	- 레디스에서 자료를 저장하는 가장 간단한 자료구조
	- 최대 512MB의 문자열 데이터를 저장
	- 어떤 바이트든(널 문자 포함) 안전하게 저장하고 처리할 수 있다. (binary-safe)
	- 키와 실제 저장되는 아이템이 1:1로 연결되는 유일한 자료구조이다.
	```
	127.0.0.1:6379> SET hello world
	OK
	
	127.0.0.1:6379> GET hello
	"world"
	```
	
	- NX 옵션: 지정한 키가 없을 때만 새로운 키를 저장
	```
	127.0.0.1:6379> SET hello newval NX
	(nil)
	```

	- XX 옵션: 지정한 키가 있을 때만 새로운 값으로 덮어쓴다.
	```
	127.0.0.1:6379> SET hello newval XX
	OK
	
	127.0.0.1:6379> GET hello
	"newval"
	```

	- 숫자 형태도 저장 가능
	- INCR: 저장된 데이터 1씩 증가, 증가된 값을 반환
	- INCRBY: 입력한 값만큼 증가
	- DECR, DECRBY 도 동일한 방식으로 동작(감소)
	```
	127.0.0.1:6379> SET counter 100
	OK
	
	127.0.0.1:6379> INCR counter
	101

	127.0.0.1:6379> INCR counter
	102
	
	127.0.0.1:6379> INCRBY counter 50
	152
	```

	- MSET, MGET: 한번에 여러 키를 조작 가능
	```
	127.0.0.1:6379> MSET a 10 b 20 c 30
	OK
	
	127.0.0.1:6379> MGET a b c
	1) "10"
	2) "20"
	3) "30"
	```



- list
	- 하나의 list에 최대 42억여개의 아이템 저장이 가능
	- 다른 배열처럼 인덱스 접근이 가능하고, 서비스에서는 스택과 큐로서 사용된다.
	- LPUSH: list 왼쪽에 데이터 추가
	- RPUSH: list 오른쪽에 데이터 추가
	- LRANGE: list 데이터 조회
	```
	127.0.0.1:6379> LPUSH mylist E
	(integer) 1
	
	127.0.0.1:6379> RPUSH mylist B
	(integer) 2
	
	127.0.0.1:6379> LPUSH mylist D A C B A
	(integer) 7
	
	// 왼쪽 첫 번째 인덱스 0, 오른쪽 첫 번째 인덱스 -1
	// 인덱스로 조회 범위 설정 가능
	127.0.0.1:6379> LRANGE mylist 0 -1
	1) "A"
	2) "B"
	3) "C"
	4) "A"
	5) "D"
	6) "E"
	7) "B"

	127.0.0.1:6379> LRANGE mylist 0 3
	1) "A"
	2) "B"
	3) "C"
	4) "A"
	```

	- LPOP: list 첫 번째 데이터를 반환하면서 동시에 삭제
	```
	127.0.0.1:6379> LPOP mylist
	"A"
	
	// 지정된 숫자만큼 반복해서 반환
	127.0.0.1:6379> LPOP mylist 2
	1) "B"
	2) "C"
	```

	- LTRIM: 시작과 끝 아이템의 인덱스를 전달받아 지정된 범위에 속하지 않은 아이템은 모두 삭제하지만 삭제되는 아이템을 반환하지는 않음
	```
	127.0.0.1:6379> LRANGE mylist 0 -1
	1) "A"
	2) "D"
	3) "E"
	4) "B"
	
	127.0.0.1:6379> LTRIM mylist 0 1
	OK
	
	127.0.0.1:6379> LTRIM mylist 0 -1
	OK
	
	127.0.0.1:6379> LRANGE mylist 0 -1
		1) "A"
		2) "D"
	```


	- LPUSH와 LTRIM를 함께 사용하면 고정된 길이의 큐를 쉽게 유지할 수 있다.
	- 아래 동작은 매번 큐의 마지막 데이터만 삭제 하므로 빠르게 처리된다. -> O(1)
	```
	127.0.0.1:6379> LPUSH logdata <data>
	
	// list에 최대 1000개의 로그 데이터를 보관
	127.0.0.1:6379> LTRIM logdata 0 999
	```


	- list의 양 끝에 데이터를 넣고 빼는 LPUSH, RPUSH, LPOP, RPOP도 O(1) 로 빠르게 처리
	- 하지만 인덱스나 데이터를 이용해 접근할 때는 O(n)으로 처리

	- LINSERT: 원하는 데이터의 앞이나 뒤에 데이터를 추가
		- BEFORE: 앞
		- AFTER: 뒤
		- 지정한 데이터가 없으면 오류
	```
	127.0.0.1:6379> LRANGE mylist 0 -1
	1) "A"
	2) "B"
	3) "C"
	4) "D"
	
	// B앞에 E추가
	127.0.0.1:6379> LINSERT BEFORE B E
	(integer) 5

	127.0.0.1:6379> LRANGE mylist 0 -1
	1) "A"
	2) "E"
	3) "B"
	4) "C"
	5) "D"
	```

	- LSET: 지정한 인덱스의 데이터를 신규 입력한 데이터로 덮어쓴다.
		- 지정한 범위를 벗어나면 오류
	- LINDEX: 원하는 인덱스의 데이터 확인
	```
	127.0.0.1:6379> LSET mylist 2 F
	OK

	127.0.0.1:6379> LRANGE mylist 0 -1
	1) "A"
	2) "E"
	3) "F"
	4) "C"
	5) "D"

	127.0.0.1:6379> LINDEX mylist 3
	"C"
	```


- hash
	- 필드-값 쌍을 가진 아이템의 집합
	- 필드는 하나의 hash 내에서 유일하다.
	- 객체를 표현하기에 적절 -> 관계형 데이터베이스의 테이블 데이터로 변환이 쉬움
	- 각 아이템마다 다른 필드를 가질 수 있다. -> 유연한 개발
	- 동적으로 다양한 필드 추가할 수 있다.

	- HSET: hash에 아이템 저장 (한 번에 여러 필드-값 쌍을 저장할 수도 있음)
	```
	127.0.0.1:6379> HSET Product:123 Name "Happy Hacing"
	(integer) 1
	
	127.0.0.1:6379> HSET Product:123 TypeID 35
	(integer) 1
	
	127.0.0.1:6379> HSET Product:123 Version 2002
	(integer) 1
	
	127.0.0.1:6379> HSET Product:234 Name "Track ball" TypeID 32
	(integer) 2
	```

	- HGET: hash 아이템 조회
	- HGETALL: hadh 내의 모든 필드-값 쌍을 차례로 반환
	```
	127.0.0.1:6379> HGET Product:123 TypeID
	"35"
	
	127.0.0.1:6379> HSET Product:234 Name TypeID
	1) "Track ball"
	2) "32"

	127.0.0.1:6379> HGETALL Product:234
	1) "Name"
	2) "Track ball"
	3) "TypeID"
	4) "32"
	```


- set
	- 정렬되지 않은 문자열의 모음
	- 하나의 set 구조 내에 중복 아이템 저장 X
	- 결합 관련 연산을 지원 (교집합, 합집합, 차집합...)

	- SADD: set에 아이템을 저장
	- SMEMBERS: set에 저장된 전체 아이템 출력
	```
	127.0.0.1:6379> SADD myset A
	(integer) 1
	
	127.0.0.1:6379> SADD myset A A B B B C D E 
	(integer) 5

	// 출력 순서는 랜덤
	127.0.0.1:6379> SMEMBERS myset
	1) "A"
	2) "B"
	3) "C"
	4) "E"
	5) "D"
	```

	- SREM: set에서 원하는 데이터를 삭제
	- SPOP: set에서 랜덤 아이템을 반환 및 삭제
	```
	127.0.0.1:6379> SREM myset A
	(integer) 1
	
	127.0.0.1:6379> SPOP myset 
	"C"
	``` 

	- SUNION: 합집합
	- SINTER: 교집합
	- SDIFF: 차집합
	```
	// set:111 - A, B, C
	// set:222 - B, C, D
	127.0.0.1:6379> SUNION set:111 set:222
	1) "A"
	2) "B"
	3) "C"
	4) "D"
	
	127.0.0.1:6379> SINTER set:111 set:222 
	5) "C"
	6) "B"

	127.0.0.1:6379> SDIFF set:111 set:222 
	7) "A"
	``` 


- sortedSet
	- 스코어 값에 따라 정렬되는 고유한 문자열의 집합
	- 모든 값은 스코어-값 쌍을 가지며 정렬되어 저장
	- 같은 스코어를 가진 아이템은 데이터의 사전 순으로 정렬
	- 중복없이 유일하게 저장 -> set과 유사
	- 각 아이템은 스코어라는 데이터에 연결 -> hash와 유사
	- 인덱스를 통해 접근 가능 -> list와 유사

> - 시간 복잡도 비교
> 	list -> O(n)
> 	sortedSet -> O(log(n)) 
> 	인덱스를 이용해 아이템에 접근할 일이 많다면 list보다 sortedSet가 유리하다.

- ZADD: sortedSet에 데이터를 저장
	```
	// 스코어-값 쌍으로 입력
	// 한번에 여러개 입력 가능
	127.0.0.1:6379> ZADD score:220 100 user:B
	(integer) 1
	
	127.0.0.1:6379> ZADD score:220 150 user:A 150 user:C 200 user:F
	(integer) 3
	``` 

	- 이미 존재하는 값은 스코어만 업데이트 된다.
		- 업데이트된 스코어로 재정렬
	- 지정한 키가 없으면 새로 생성
	- 키가 이미 존재하지만 sortedSet이 아니면 오류
	- 스코어는 64비트 실수 형태
	
	- ZADD의 커맨드 옵션
		- XX: 아이템이 존재할 때만 스코어를 업데이트
		- NX: 아이템이 존재하지 않을 때만 신규 삽입, 스코어 업데이트X
		- LT: 해당 스코어 < 기존 스코어 일 때 업데이트, 존재하지 않으면 신규 삽입
		- GT: 해당 스코어 > 기존 스코어 일 때 업데이트, 존재하지 않으면 신규 삽입

	- ZRANGE: sortedSet에 저장된 데이터 조회
	```
	// 항상 start와 stop 범위를 입력해야함
	127.0.0.1:6379> ZRANGE key start stop [BYSCORE | BYLEX] [REV] [LIMIT offset count] [WITHSCORE]
	``` 

- 인덱스로 데이터 조회
	- ZRANGE는 인덱스를 기반으로 데이터를 조회하므로 start와 stop에 검색하고자 하는 첫 번째와 마지막 인덱스를 전달한다.
	- 음수 인덱스 사용 가능
	```
	// 항상 start와 stop 범위를 입력해야함
	127.0.0.1:6379> ZRANGE score:220 1 3 WITHSCORES
	1) "user:A"
	2) "150"
	3) "user:C"
	4) "150"
	5) "user:F"
	6) "200"

	// REV: 역순 출력
	127.0.0.1:6379> ZRANGE score:220 1 3 WITHSCORES REV
	1) "user:C"
	2) "150"
	3) "user:A"
	4) "150"
	5) "user:B"
	6) "100"
	``` 

- 스코어로 데이터 조회
	- BYSCORE: 스코어를 이용해 데이터 조회
	```
	127.0.0.1:6379> ZRANGE score: 220 100 150 BYSCORE WITHSCORES
	
	// (를 앞에 붙이면 해당 스코어를 포함하지 않는 값 조회
	// (: 포함, [: 불포함
	127.0.0.1:6379> ZRANGE score: 220 (100 150 BYSCORE WITHSCORES
	```
  
	- +inf: 최댓값
	- -inf: 최솟값

	```
	// 전체 조회
	127.0.0.1:6379> ZRANGE score: 220 -inf +inf BYSCORE WITHSCORES
	
	// 200~최댓값
	127.0.0.1:6379> ZRANGE score: 220 200 +inf BYSCORE WITHSCORES
	
	// 최댓값~200
	// REV는 역순 출력이므로 큰 값을 start 값으로
	127.0.0.1:6379> ZRANGE score: 220 +inf 200 BYSCORE WITHSCORES REV
	```

- 사전순으로 데이터 조회
	- BYREX: 스코어가 같을 때 사전식 순서로 특정 데이터를 조회
	- 문자열은 ASCII 코드를 기준으로 하기 때문에 한글도 사전식 정렬이나 조회가 가능
	```
	// 전체 조회
	127.0.0.1:6379> ZRANGE mySortedSet - + BYREX
	
	// ( 또는 [를 항상 포함
	127.0.0.1:6379> ZRANGE mySortedSet (b (f BYREX
	```

- 비트맵
	- string 자료구조에 비트 연산이 가능하도록 확장한 형태
	- string = 2^32 비트를 가지고 있는 비트맵 형태    
	<br/>
	- SETBIT: 비트 저장
	- GETBIT: 저장된 비트 조회
	- BITFIELD: 한 번에 여러 비트 저장
	```
	127.0.0.1:6379> SETBIT mybitmap 2 1
	(integer) 1
	
	127.0.0.1:6379> GETBIT mybitmap 2 
	(integer) 1
	
	127.0.0.1:6379> BITFIELD mybitmap SET u1 6 1 SET u1 10 1 SET u1 14 1
	1) (integer) 0
	2) (integer) 0
	3) (integer) 0
	```

	- BITCOUNT: 1로 설정된 비트의 갯수 카운팅
	```
	127.0.0.1:6379> BITCOUNT mybitmap
	(integer) 4
	```
  
- Hyperloglog
	- 집합의 원소 갯수 = 카디널리티를 추정할 수 있는 자료구조
	- 대량 데이터에서 중복되지 않는 고유한 값을 집계할 때 유용하다.
	- 입력되는 데이터를 자체적으로 변경해서 저장
		- 데이터 갯수에 구애받지 않고 일정한 메모리를 유지
		- set은 중복을 피하기 위해 모든 데이터를 기억하므로 데이터가 많아지면 메모리 사용량이 많아진다.
	- 최대 12KB
	<br/>
	- PFADD: 아이템 저장
	- PFCOUNT: 저장된 아이템의 갯수 = 추정 카디널리티
	```
	127.0.0.1:6379> PFADD members 123
	(integer) 1
	
	127.0.0.1:6379> PFADD members 500
	(integer) 1
	
	127.0.0.1:6379> PFADD members 12
	(integer) 1
	
	127.0.0.1:6379> PFCOUNT members
	(integer) 3
	```
  

- Geospatial
	- 위도-경도 데이터 쌍으로 지리 데이터를 저장
	- 내부적으로 sorted set으로 저장
		- sorted set 의 커맨드 옵션 사용 가능
	- 하나의 자료구조 안에 키는 중복될 수 없음
	```
	// 저장
	GEOADD <key> 경도 위도 member
	
	// 조회
	 GEOPOS <key> member
	
	// 두 아이템 사이 거리
	// KM - 거리 단위(킬로미터)
	GEODIST <key> member1 member2 KM
	```

- stream
	- 레디스를 메세지 브로커로 쓸 수 있게하는 자료구조
	- 카프카의 영향을 받음


## 레디스에서 키를 관리하는 법

- 키의 자동 생성과 삭제
	- stream이나 hash, set 등 하나의 키가 여러 아이템을 갖고 있는 자료구조의 경우, 키는 알아서 생성되고 삭제된다.
		<br/>
	- 키 생성과 삭제의 공통적인 규칙
		* 키가 존재하지 않을 때 아이템을 넣으면 아이템을 넣기전에 빈 자료구조를 생성한다.
		```
		127.0.0.1:6379> DEL mylist
		(integer) 1
		
		// mylist 라는 키가 없어도 데이터를 넣으면 자동 생성
		127.0.0.1:6379> LPUSH mylist 1 2 3
		(integer) 3
		
		127.0.0.1:6379> SET hello world
		OK
		
		127.0.0.1:6379> LPUSH hello 1 2 3
		(error) WRONGTYPE Operation against a key holding the wrong kind of value
		
		127.0.0.1:6379> TYPE hello
		string
		```
		
		- 모든 아이템을 삭제하면 키도 삭제된다. (stream 예외)
		```
		127.0.0.1:6379> LPUSH mylist 1 2 3
		(integer) 3
		
		127.0.0.1:6379> EXISTS mylist
		(integer) 1
		
		127.0.0.1:6379> LPOP mylist
		"3"
		
		127.0.0.1:6379> LPOP mylist
		"2"
		
		127.0.0.1:6379> LPOP mylist
		"1"

		// 아이템을 모두 삭제하면 키가 사라짐
		127.0.0.1:6379> EXISTS mylist
		(integer) 0
		```

		- 키가 없는 상황에서 키 삭제, 아이템 삭제, 자료구조 크기 조회같은 읽기 전용 커맨드를 실행하면 에러대신 키가 있으나 아이템이 없는 것처럼 동작

		```
		127.0.0.1:6379> DEL mylist
		(integer) 0
		
		127.0.0.1:6379> LLEN mylist
		(integer) 0
		
		127.0.0.1:6379> LPOP mylist
		(nil)
		```

- 키와 관련된 커맨드
	- 자료 구조와 관련없이 공통적으로 사용 가능
		<br/>
	- 키의 조회
		- EXISTS: 키가 존재하는지 확인
		```
		// 존재하면 1 아니면 0
		EXISTS key [key...]
		
		127.0.0.1:6379> SET hello world
		OK

		127.0.0.1:6379> EXISTS hello
		(integer) 1
		
		127.0.0.1:6379> EXISTS world
		(integer) 0		
		```


		- KEYS: 레디스에 저장된 모든 키를 조회
		```
		// 패턴에 해당되는 모든 키를 list 형태로 반환
		KEYS pattern
		```
  
		- 글롭 패턴 스타일로 동작
			- h?llo에는 hello, hallo가 매칭될 수 있다.
			-  hllo에는 hllo, heeeello가 매칭될 수 있다.
			-  h[ae]llo에는 hello, hallo가 매칭될 수 있지만, hillo는 매칭되지 않는다.
			-  h[^e]llo에는 hallo, hbllo가 매칭될 수 있지만, hello는 매칭되지 않는다.
			- h[a-b]llo에는 hallo, hbllo만 매칭될 수 있다.

> 	- KEYS 의 위험성
> 	KEYS는 레디스에 저장된 모든 키를 반환한다. 
> 	레디스는 싱글스레드로 동작하기 때문에 작업이 오래 걸리게 되면 그동안 마스터는 다른 작업을 수행할 수 없기 때문에 클라이언트에 나중에 들어온 요청으로 대기열이 늘어나거나 모니터링 도구가 마스터 노드에 보낸 helth check에 응답하지 못해 페일오버가 발생할 수 있다.

- SCAN: 커서를 기반으로 특정 범위의 키만 조회 가능
```
SCAN cursor [MATCH pattern] [COUNT count] [TYPE type]

127.0.0.1:6379> SCAN 0
1) "7" // 다음 SCAN 시 사용할 인수
2)  1) "mybitmap"
    2) "Product:123"
    3) "b"
    4) "score:220"
    5) "a"
    6) "myset"
    7) "Product:234"
    8) "members"
    9) "c"
   10) "hello"

127.0.0.1:6379> SCAN 7
1) "0" // 레디스의 모든 키를 반환해서 더 이상 검색할 키가 없음
2) (empty array)
```

- MATCH 옵션 사용 시, KEYS에서처럼 입력한 패턴에 맞는 키 값을 조회
```
127.0.0.1:6379> KEYS *my*
1) "mybitmap"
2) "myset"
```

  

- SORT: list, set, sorted set에서만 사용할 수 있는 커맨드로 키 내부의 아이템을 정렬해 반환
	```
	SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC | DESC] [ALPHA] [STORE destination]
	```
  

	- RENAME / RENAMENX: 키의 이름을 변경
	```
	RENAME key newkey
	// 변경할 키가 존재하지 않을 때만 동작
	RENAMENX key newkey 
	```
	
	- COPY: source에 지정된 키를 destination에 복사
	```
	COPY source destination [DB destination-db] [REPLACE]
	```

	- TYPE: 지정한 키의 자료 구조 타입을 반환
	```
	TYPE key
	```

	- OBJECT: 키에 대한 상세 정보를 반환
	```
	// subcommand - ENCODING, IDLETIME
	OBJECT <subcommand> [<arg> [value] [opt] ...]
	```

  

  

- 키의 삭제
	- FLUSHALL: 레디스에 저장된 모든 키를 삭제
	```
	FLUSHALL [ASYNC | SYNC]
	```
	- SYNC: 
		- 모든 데이터가 삭제된 경우만 OK 반환
		- 커맨드가 실행중일 때는 다른 응답을 처리 못함
	- ASYNC: 
		- flush를 백그라운드로 실행
		- 커맨드가 수행됐을 때 존재했던 키만 삭제
		- flush중 새로 생성된 키는 삭제되지 않음

	- DEL: 키와 키에 저장된 데이터 모두 삭제
		- 기본적으로 동기적으로 작동
	```
	DEL key [key ...]
	```

	- UNLINK: DEL과 비슷하게 키와 데이터를 삭제하는 커맨드
		- 백그라운드에서 다른 스레드에 의해 처리
		- 우선 키와 연결된 데이터의 연결을 끊는다.
	```
	UNLINK key [key ...]
	```

 > - set, sorted set과 같이 하나의 키에 여러 개의 데이터가 연결된 자료구조의 경우 키를 DEL로 삭제하면 어떤 영향이 갈 지 모른다.
> - lazyfree-lazy user-del 옵션이 yes 일 경우 모든 DEL은 UNLINK로 동작해서 백그라운드로 키를 삭제 -> 옵션 기본값: no


- 키의 만료 시간
	- EXPIRE: 키가 만료될 시간을 초단위로 정의할 수 있다.
		-   NX: 해당 키에 만료 시간이 정의돼 있지 않을 경우에만 커맨드 수행
		-   XX: 해당 키에 만료 시간이 정의돼 있을 때에만 커맨드 수행
		-   GT: 현재 키가 가지고 있는 만료 시간보다 새로 입력한 초가 더 클 때에만 수행
		-   LT: 현재 키가 가지고 있는 만료 시간보다 새로 입력한 초가 더 작을 때에만 수행
	```
	EXPIRE key seconds [ NX | X | GT | LT]
	```

	- EXPIREAT: 키가 특정 유닉스 타임스탬프에 만료될 수 있도록 키의 만료 시간을 직접 지정
		- 옵션은 EXPIRE와 동일
	```
	EXPIREAT key unix-time-seconds [NX XX | GT | LT]
	```

	- EXPIRETIME: 키가 삭제되는 유닉스 타임스탬프를 초 단위로 반환
	```
	EXPIRETIME key
	```

	- TTL: 키가 몇 초 뒤에 만료되는지 반환
	```
	// -1: key가 존재하지만 만료 시간 설정X
	// -2: key가 없음
	TTL key
	```
  
