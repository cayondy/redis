02. 레디스 시작하기

## MacOS에 레디스 설치하기

- Homebrew를 사용해서 설치하기
```
// 설치
$ brew install redis

// 실행
$ brew services start redis
```

## 레디스 실행하기

- 프로세스 시작과 종료
	- 아래 방법은 demonize를 yes 로 설정해야한다.

```
// 프로세스 시작
$ bin/redis-server redis.conf

// 프로세스 종료
$ bin/redis-cli shutdown
```


- 레디스 접속하기
	- redis-cli(레디스 클라이언트)는 bin 디렉토리안에 존재한다.
	- 원하는 경로를 PATH로 지정해두면 어느 위치에서든 바로 접근할 수 있다.
```
// 디렉토리 경로 설정
$ expert PATH=$PATH:/home/centos/redis/bin

// 레디스 서버 접근
redis-cli -h <ip주소> -p <port> -a <password>
```


	
	- ip 주소 기본값: 127.0.0.1
	- port 기본값: 6379
	- requirepass에 패스워드 설정해둔 경우만 -a <password> 필요



- 데이터 저장과 조회
```
$ redis-cli
127.0.0.1:6379> SET hello world // hello:키 world:밸류 저장
OK

> GET hello
"world"
```

- 파이썬에서 사용하기
	- redis-py 클라이언트 사용
```
// 클라이언트 다운로드
$ pip install redis

>>> import redis
>>> r = redis.Redis(host='localhost', port=6379. db=0)
>>> r.get('hello')
b'world'
