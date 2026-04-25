# Boost.Asio

## 1. Boost.Asio란?

Boost.Asio는 C++로 작성된 **비동기 입출력(I/O) 라이브러리**입니다. (Asio = **A**synchronous **I**nput **O**utput)  
네트워크 통신, 타이머, 시리얼 포트 등 다양한 I/O 작업을 비동기로 처리할 수 있게 해줍니다.

채팅 서버, HTTP 서버, 게임 서버처럼  
다수의 동시 연결을 다루는 프로그램을 만들 때 사실상 표준입니다.

<br>

## 왜 비동기 I/O가 필요한가요?

전통적인 블로킹 I/O로 서버를 개발하면 각 요청마다 응답이 올 때까지 서버가 멈추기 때문에  
CPU 자원을 효율적으로 사용하지 못하고 비효율적으로 동작하게 됩니다.

```cpp
// 블로킹 방식 (안 좋은 예)
while (true) {
    socket s = accept(listenSock);  // 연결 들어올 때까지 멈춤
    char buf[1024];
    recv(...);                      // 데이터 올 때까지 멈춤
    // 한 클라이언트 처리 동안 다른 클라이언트는 대기
}
```

이를 해결하기 위해 **비동기 I/O** 방식으로  
적은 수의 스레드(보통 CPU 코어 수)로 수만 개 연결을 처리합니다.

이것이 Asio가 제공하는 모델입니다.

<br>

## Boost.Asio는 왜 사용하나요? (장단점)

### 장점

- 크로스 플랫폼 (한 코드가 IOCP/epoll/kqueue 위에서 다 돌아감)
- 모던 C++ 인터페이스 (람다, C++20 지원)
- Boost 생태계와 자연스럽게 결합
- C++ 표준 위원회의 표준화 베이스

<br>

### 단점

- 다른 비동기 모델 이해가 있어야 쉬움
- 헤더가 무거워 컴파일 시간 증가
- 콜백 체인 디버깅이 까다로움

<br>

> **"성능과 유지보수성의 균형이 가장 좋은 선택"**  
> 정말 극한 성능이 필요하면 IOCP등 으로 가지만, 보통의 서버는 Asio로 충분합니다.

<br>

## io_context와 io_run

### io_context (이벤트 루프)

모든 비동기 작업이 등록되는 중앙 객체입니다.  
개념적으로 **할 일 큐 + OS 이벤트 처리기**입니다.

```cpp
asio::io_context io;
```

이 한 줄로 빈 이벤트 루프 객체가 생깁니다.  
또한 호출자가 run()을 부를 때까지 대기 상태입니다.

<br>

### io.run() (이벤트 루프 실행)

```cpp
io.run();
```

<br>
 
호출한 스레드가 이벤트 루프 워커가 됩니다.
 
```cpp
while (할 일 존재) {
    // 1. OS로부터 완료/준비 이벤트 수집 (IOCP/epoll)
    // 2. 완료된 작업의 핸들러를 큐에서 꺼냄
    // 3. 현재 스레드에서 핸들러 실행
}
```

등록된 작업이 모두 끝나면 run()은 리턴합니다.  
서버는 보통 핸들러 안에서 다음 작업을 다시 등록하므로 무한히 돌게 됩니다.

<br>

### 멀티스레드에서 워커 풀

io_context는 thread-safe해서 같은 객체에 대해  
여러 스레드가 동시에 run()을 호출할 수 있습니다.

> 이것이 **표준 스레드 풀 패턴**입니다.

<br>

### Linux-Asio-ChatServer 코드 일부 예시

```cpp
asio::io_context io;
Server server(io, port);  // accept 작업이 io에 등록됨 (아직 안 돔)

unsigned int n = std::thread::hardware_concurrency();
if (n == 0) n = 4;

std::vector<std::thread> workers;
for (unsigned int i = 0; i < n; ++i) {
    workers.emplace_back([&io]() { io.run(); });
}

for (auto& t : workers) t.join();
```

n개 스레드가 같은 io에 대해 run()을 호출하면,  
Asio가 핸들러를 노는 스레드에 분배합니다.

> N개 핸들러를 N개 스레드가 동시 실행하는 구조가 됩니다!

 <br>

## async_accept

### Linux-Asio-ChatServer 코드 일부 예시

```cpp
class Server {
public:
    Server(asio::io_context& io, uint16_t port)
        : acceptor_(io, tcp::endpoint(tcp::v4(), port))
        , mongo_("mongodb://127.0.0.1:27017", "chat", "logs")
        , room_(io, mongo_)
    {
        mongo_.Start();
        Accept();
        std::cout << "[Server] Listening on 0.0.0.0:" << port << "\n";
    }

private:
    void Accept() {
        acceptor_.async_accept([this](boost::system::error_code ec, tcp::socket socket) {
            if (!ec) {
                auto s = std::make_shared<Session>(std::move(socket), room_, mongo_);
                room_.join(s);
                s->Start();
            }
            Accept();  // 다음 연결을 위해 재등록
        });
    }

    tcp::acceptor acceptor_;
    MongodbManager mongo_;
    SessionManager room_;
};
```

<br>

### 1. 동작 흐름

1. async_accept(handler) 호출 → 연결 들어오면 핸들러 호출해달라는 요청을 io_context에 등록만 하고 즉시 리턴
2. 워커 스레드가 io.run()을 돌다가, OS로부터 연결 완료 알림을 받음
3. 등록해둔 핸들러를 워커 스레드에서 실행
4. 핸들러 안에서 Accept()를 다시 호출 → 다음 연결을 위한 작업 재등록
5. 반복

<br>

### 2. socket 파라미터는 어디서 왔나요? (Windows IOCP 비교 예시)

async_accept의 핸들러는 tcp::socket socket을 인자로 받는데,  
이 소켓은 **Asio가 내부적으로 미리 생성해서 OS에 등록**해 둔 것입니다.

사용자는 새 소켓을 만들 필요 없이 연결 완료된 상태로 받기만 하면 됩니다.

<br>

#### Windows IOCP와 비교

| 구분                   | Boost.Asio               | IOCP                               |
| ---------------------- | ------------------------ | ---------------------------------- |
| 소켓 생성 주체         | 내부적으로 생성          | 사용자가 직접 생성                 |
| 소켓 등록              | 생성된 소켓 자동 OS 등록 | 사용자가 생성한 소켓으로 직접 등록 |
| connect 요청 응답 통지 | 내부적으로 처리          | 소켓 연결 정보 채워서 통지         |

---

#### 조심할 점

Server 생성자에서 async_accept가 호출되어 작업이 등록되지만,
**실제로는 처리되지 않습니다**

위 코드 예시처럼 워커 스레드가 등장해서 run()을 시작해야  
비로소 OS와 통신하며 핸들러가 호출됩니다. (등록과 실행은 분리)

---

<br>

## async_read / async_write

### shared_from_this 패턴

비동기 핸들러가 호출되는 시점까지 Session 객체가 살아있어야 합니다.  
람다에 self = shared_from_this()를 캡처하면,  
**핸들러가 살아있는 한 Session도 살아있다**는 게 자동 보장됩니다.

핸들러가 끝나고 람다가 소멸되면 self도 함께 소멸되고,  
그게 마지막 참조였다면 Session 자체도 소멸되어 자동으로 정리됩니다.

<br>

> RAII와 shared_ptr 조합으로
> IOCP 시절의 직접 관리하던 코드가 거의 사라집니다.

<br>

### Linux-Asio-ChatServer 코드 예시 (Read)

```cpp
void Read() {
    auto self = shared_from_this();

    asio::async_read_until(socket_, inbuf_, '\n',
        asio::bind_executor(strand_,
            [this, self](boost::system::error_code ec, std::size_t) {
                if (ec) {
                    if (ec == asio::error::operation_aborted) {
                        // 서버측 취소 (의도된 취소)
                    }
                    else if (ec == asio::error::eof || ec == asio::error::connection_reset) {
                        // 클라이언트가 정상적으로 연결을 끊었거나, 연결이 리셋된 경우
                        std::cout << "[Session] " << user_name_ << " disconnected: " << ec.message() << "\n";
                    }
                    else {
                        // 예상 못한 에러
                        std::cerr << "[Session] Unexpected error in " << user_name_ << ": " << ec.message() << "\n";
                    }
                    room_.leave(self);
                    return;
                }

                // 정상 처리 (메시지 파싱 후 다시 Read 호출)
                // 채팅 타입별 분기 처리

                Read();  // 다음 메시지를 받기 위해 재등록
            }
        )
    );
}
```

<br>

### Linux-Asio-ChatServer 코드 예시 (Write)

```cpp
void Write() {
    auto self = shared_from_this();
    if (outbox_.empty()) { writing_ = false; return; }

    auto msg = outbox_.front();
    asio::async_write(socket_, asio::buffer(*msg),
        asio::bind_executor(strand_,
            [this, self](boost::system::error_code ec, std::size_t) {
                if (ec) {
                    room_.leave(self);
                    return;
                }
                outbox_.pop_front();
                Write();  // 남은 메시지가 있으면 계속 보냄
            }
        )
    );
}
```

---

### 유저 접속 끊김/종료 감지 방법

Asio는 OS별 끊김 신호(FIN, RST, 시스템 에러 등)를 모두  
**boost::system::error_code 한 곳으로 통합**해서 핸들러에 던져줍니다.

| IOCP 케이스                 | Asio에서의 ec 값                |
| --------------------------- | ------------------------------- |
| bytes == 0 (FIN, 정상 종료) | asio::error::eof                |
| WSAECONNRESET (강제 종료)   | asio::error::connection_reset   |
| WSAECONNABORTED             | asio::error::connection_aborted |
| 우리가 close/cancel 호출    | asio::error::operation_aborted  |

> Windows든 Linux든 macOS든 **ec 한 번 체크로 모든 끊김 처리 가능**

<br>

#### ec 뜨면 무조건 leave가 정석

TCP 스트림은 한 번 망가지면 그 세션을 다시 쓰기 어렵습니다.  
재시도 가능한 저수준 에러는 Asio가 내부에서 자체 처리하므로,  
**핸들러까지 올라온 ec는 사실상 회복 불가능한 상황**입니다.

따라서 'if (ec) leave(self); return;' 패턴이 사실상 표준입니다.  
운영 시에는 ec 종류별로 로그 레벨만 분기하면 충분합니다.

---

<br>

## 멀티스레드와 strand

### 1. 데이터 레이스 위험

워커 스레드가 여러 개면  
**같은 Session의 핸들러가 서로 다른 스레드에서 동시에 실행**될 수 있습니다.

```
스레드 1: Session A의 read 핸들러 실행 중 (buffer_ 읽기 중)
스레드 2: Session A의 write 핸들러 실행 중 (buffer_ 수정 중)
→ 데이터 레이스
```

<br>

### 2. 해결책 → strand

strand는 "이 strand에 묶인 핸들러들은 절대 동시 실행되지 않는다"를 보장하는 객체입니다.  
같은 strand에 묶인 핸들러들은 어느 스레드가 잡든 항상 한 번에 하나씩만 실행됩니다.

> 멀티스레드 io_context를 쓸 때  
> **세션마다 strand를 두는 것이 거의 표준 패턴**입니다.

<br>

### Linux-Asio-ChatServer 코드 예시 (Session strand)

```cpp
class Session : public std::enable_shared_from_this<Session> {
public:
    Session(tcp::socket socket, SessionManager& room, MongodbManager& mongo)
        : socket_(std::move(socket))
        , room_(room)
        , mongo_(mongo)
        , strand_(asio::make_strand(socket_.get_executor())) {}  // 세션 전용 strand

    void Start() {
        asio::dispatch(strand_, [self = shared_from_this()]() {
            self->Read();
        });
    }

    void Deliver(const std::shared_ptr<const std::string>& msg) {
        asio::post(strand_, [self = shared_from_this(), msg]() {
            self->outbox_.push_back(msg);
            if (self->writing_) return;
            self->writing_ = true;
            self->Write();
        });
    }

private:
    // 세션 전용 strand
    asio::strand<tcp::socket::executor_type> strand_;

    // ... 다른 멤버들
};
```

> 각 비동기 작업에 `asio::bind_executor(strand_, ...)`로 묶으면  
> 그 세션의 모든 핸들러가 직렬화됩니다.

<br>

### Linux-Asio-ChatServer 코드 예시 (SessionManager strand)

세션 전체를 관리하는 SessionManager도 자체 strand로 컨테이너 보호

```cpp
class SessionManager {
public:
    using executor_type = asio::io_context::executor_type;
    using strand_type   = asio::strand<executor_type>;

    SessionManager(asio::io_context& io, MongodbManager& mongo)
        : strand_(asio::make_strand(io)), mongo_(mongo) {}

    void join(const std::shared_ptr<Session>& s);
    void leave(const std::shared_ptr<Session>& s);
    void broadcast(const std::shared_ptr<const std::string>& msg);

private:
    strand_type strand_;
    MongodbManager& mongo_;

    std::set<std::shared_ptr<Session>> sessions_;
    std::unordered_map<std::string, std::shared_ptr<Session>> conn_user_sessions_;
};

// 모든 컨테이너 접근을 strand_로 직렬화
void SessionManager::join(const std::shared_ptr<Session>& s) {
    asio::post(strand_, [this, s]() {
        sessions_.insert(s);
    });
}

void SessionManager::leave(const std::shared_ptr<Session>& s) {
    asio::post(strand_, [this, s]() {
        conn_user_sessions_.erase(s->GetUserName());
        sessions_.erase(s);
    });
}

void SessionManager::broadcast(const std::shared_ptr<const std::string>& msg) {
    asio::post(strand_, [this, msg]() {
        // 브로드캐스트 중에 세션이 join/leave 하더라도 안전하게 처리하기 위해
        // tempSession을 만들어서 현재 세션 목록을 복사한 후, 그 복사본을 순회
        auto tempSession = sessions_;

        for (auto& s : tempSession) {
            s->Deliver(msg);
        }
    });
}
```

> 락(mutex) 없이도 strand 하나로 데이터 일관성 보장

<br>

### 단일 스레드면 strand 불필요

io.run()을 한 스레드에서만 부른다면  
모든 핸들러가 그 한 스레드에서 직렬 실행되므로 strand가 필요 없습니다.

> 멀티스레드로 갈 때만 사용 권장

<br>

## 내부 동작 원리 (OS별로 어떻게 도는가)

### proactor vs reactor

| 모델     | 의미                                                | 대표 OS      |
| -------- | --------------------------------------------------- | ------------ |
| proactor | OS가 작업을 끝까지 수행 후 완료 알림                | Windows IOCP |
| reactor  | OS가 "준비됨" 알림만 주고, 실제 I/O는 사용자가 호출 | Linux epoll  |

> Asio는 두 모델을 통일해서
> 사용자에게는 항상 proactor 인터페이스(완료 핸들러)를 노출합니다.

<br>

### IOCP (proactor, 비동기)

```
[Asio가 하는 일 (IOCP 시점)]
1. AcceptEx()로 작업 등록 (소켓도 미리 만들어서 같이 넘김)
2. 워커 스레드는 GetQueuedCompletionStatus()로 대기
3. 클라이언트 connect → OS가 3-way handshake 처리
4. OS가 IOCP 완료 큐에 완료 패킷 push
5. GetQueuedCompletionStatus가 깨어남 → 핸들러 실행
```

> OS가 **작업 결과(받은 데이터, 연결된 소켓 등)까지 다 만들어서** 알려주는 진짜 비동기

<br>

### Linux: epoll (reactor를 비동기처럼 에뮬레이션)

```
[Asio가 하는 일 (Linux 시점)]
1. accept fd를 epoll에 등록 (EPOLLIN으로 readable 감시)
2. 워커 스레드는 epoll_wait()로 대기
3. 클라이언트 connect → OS가 fd를 readable로 표시
4. epoll_wait가 깨어남
5. Asio가 그제서야 accept() 호출 (즉시 리턴됨, 큐에 쌓여있으므로)
6. 받아낸 fd를 tcp::socket으로 포장
7. 핸들러 실행
```

> OS는 "지금 호출하면 안 막힌다"만 알려주고, 실제 accept/read는 Asio가 직접 호출

<br>
 
### 핵심 (사용자 코드는 동일)
 
같은 acceptor_.async_accept(handler) 한 줄이   
OS에 따라 완전히 다른 시스템콜로 변환됩니다.

이게 Asio 추상화의 가장 큰 가치이며,  
크로스 플랫폼을 지원해주는 장점을 부여합니다.

<br>

## 📌 핵심 정리

- io_context는 이벤트 루프, run()을 호출한 스레드가 워커가 됨
- 비동기 작업은 등록만 하고 즉시 리턴, 실제 실행은 워커 스레드에서
- 멀티스레드 환경에선 strand로 같은 세션의 핸들러 직렬화
- 끊김/에러는 ec 한 곳으로 통합 (ec면 leave가 정석)
- 사용자 코드는 OS 독립, 내부적으로 IOCP/epoll/kqueue로 자동 변환

<br>

> Asio를 이해하면 OS별 비동기 모델 차이를 신경쓰지 않고 안전한 크로스 플랫폼 서버를 만들 수 있습니다.
