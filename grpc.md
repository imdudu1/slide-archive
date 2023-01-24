---
theme: geist
highlighter: shiki
title: gRPC 톺아보기 
info: 'HTTP API에 익숙한 팀원들에게 gRPC를 소개합니다.'
colorSchema: 'light'
fonts:
  mono: 'Noto Sans Mono'
---

# <span class='text-red-400'>Google</span>에서 만든 <br /><span class='text-blue-400'>gRPC</span> 톺아보기

<span class='text-green-400 font-semibold'>인프랩</span> 요한(@yohanio)

---

# gRPC 소개

<div class='relative mt-8'>
  <img class='h-80 rounded mx-auto' src='https://grpc.io/img/logos/grpc-icon-color.png' />
</div>

---

# gRPC 소개

<p class='text-xl my-8'>
물리적으로 떨어진 두 서버가 통신할 수 있는 <strong>프로토콜</strong><br />
<strong>동일한 환경(OS, 개발 언어, 등등)이 아니라도</strong> 서로 통신할 수 있음
</p>
<div class='relative'>
    <p class='text-5xl mb-8 slidev-vclick-target' v-click='1'>
        <div class='absolute'>
            <span v-click-hide='2' class='text-red-700 font-semibold'>HTTP API === gRPC</span>
        </div>
        <div class='absolute'>
            <span v-click='2' class='text-green-400 font-semibold'>HTTP API !== gRPC</span>
        </div>
    </p>
</div>

---

# gPRC 소개

<div class='flex mt-8'>
<div class='flex-1'>
    <p class='text-4xl font-bold'>HTTP/2</p>
    <ul class='list-disc'>
        <li class='text-xl text-green-400 font-semibold'>Binary framing layer</li>
        <li class='text-xl'>Full multiplexing</li>
        <li class='text-xl'>Flow control</li>
        <li class='text-xl text-green-400 font-semibold'>Header compression</li>
        <li class='text-xl'>Stream prioritization</li>
        <li class='text-xl'>Server push</li>
    </ul>
</div>
<div class='flex-2'>
    <img class=' rounded mx-auto' src='https://images.ctfassets.net/ee3ypdtck0rk/51ED5hLlKFbEPwUuLQ8RcR/ecdb4dab92d552050eb24e46e6ff4717/8.gif?w=569&h=314&q=50&fm=webp'>
</div>
</div>

---

# gRPC 소개

<div class='flex mt-8'>
<div class='flex-1'>
    <p class='text-4xl font-bold'>Protocol Buffers</p>
    <p class='text-xl my-8'>
    구글에서 개발 및 공개한 <strong>직렬화 데이터 구조</strong>로 <strong>다양한 언어를 지원</strong>하며 직렬화 속도가 빠름
    </p>
    <p class='text-xl my-8'>
    <strong>타입 기반</strong> 데이터 구조를 선언하면 선언된 구조를 기반으로 <strong>각 언어에 맞는 데이터 타입을 자동으로 생성</strong>할 수 있다.
    </p>
    <p class='text-xl my-8'>
    분리된 서비스들 간에 필요한 데이터 구조를 하나의 저장소에서 코드로 관리할 수 있어 <strong>SSOT(단일 진실 공급원, Single source of truth)</strong>를 지킬 수 있다.
    </p>
</div>
<div class='flex-2 m-auto'>
    <img class='w-100 rounded mx-auto' src='https://www.freecodecamp.org/news/content/images/2020/05/unnamed-1.png'>
</div>
</div>

---

# Proto Buffers

<h4 class='mb-2 mt-8 inline-block font-mono'>
session.proto
</h4>

```proto {5-11|13-16}
syntax = "proto2";

package session;

service SessionService {
  rpc FindOne (SidRequest) returns (Session) {}
}

message SidRequest {
   required string sid = 1;
}

message Session {
  required int32 id = 1;
  required string name = 2;
}
```

---

# Proto Buffers

<h4 class='mb-2 mt-8 inline-block font-mono'>
session.ts
</h4>

```ts {0-9|10-16}
export interface SidRequest {
  sid: string;
}

export interface Session {
  id: number;
  name: string;
}

export interface SessionServiceClient {
  findOne(request: SidRequest): Observable<Session>;
}

export interface SessionServiceController {
  findOne(request: SidRequest): Promise<Session> | Observable<Session> | Session;
}
```

---

# gRPC in Nest.js

<p class='text-xl'>
gRPC 서버 측 설정
</p>

<p class='mb-2 mt-8 inline-block font-mono font-semibold'>
main.ts
</p>

```ts
  app.connectMicroservice<MicroserviceOptions>({
    transport: Transport.GRPC,
    options: {
      package: 'session',
      protoPath: join(__dirname, '/proto/session.proto'),
      url: 'localhost:3001',
    },
  });

  ...중략...

  await app.startAllMicroservices();
```

---

# gRPC in Nest.js

<p class='text-xl'>
gRPC 서버 측 설정
</p>

<p class='mb-2 mt-8 inline-block font-mono font-semibold'>
SessionGRPCController.ts
</p>

```ts {2|3|all}
  @Controller('sample')
  @SessionServiceControllerMethods()
  export class SampleController implements SessionServiceController {
    findOne(request: SidRequest): Promise<Session> | Observable<Session> | Session {
      return {
        id: 1,
        name: 'test',
      };
    }
  }
```

---

# gRPC in Nest.js

<p class='text-xl'>
gRPC 클라이언트 측 설정
</p>

<p class='mb-2 mt-8 inline-block font-mono font-semibold'>
SessionExternalModule.ts
</p>

```ts {all|5|8|all}
imports: [
  ClientsModule.register([
    {
      name: 'HERO_PACKAGE',
      transport: Transport.GRPC,
      options: {
        package: 'hero',
        protoPath: join(__dirname, 'hero/hero.proto'),
      },
    },
  ]),
];
```

---

# gRPC in Nest.js

<p class='text-xl'>
gRPC 클라이언트 측 설정
</p>

<p class='mb-2 mt-8 inline-block font-mono font-semibold'>
SessionExternalService.ts
</p>

```ts {all|5|8|12|all}
@Injectable()
export class SessionExternalService implements OnModuleInit {
  private heroesService: HeroesService;

  constructor(@Inject('HERO_PACKAGE') private client: ClientGrpc) {}

  onModuleInit() {
    this.heroesService = this.client.getService<HeroesService>('HeroesService');
  }

  getHero(): Observable<string> {
    return this.heroesService.findOne({ id: 1 });
  }
}
```
