##### 1. 기능 중심
```
project/
  ├── @types/
  ├── config/
  ├── logs/
  ├── public/
  ├── src/
  |    ├── constants/
  |    ├── controllers/
  |    ├── models/
  |    ├── routes/
  |    ├── repository/
  |    ├── services/
  |    ├── middlewares/
  |    ├── utils/
  |    ├── helper/
  |    └── app.ts
  ├── views/
  └── .env
```

2. 도메인 중심
```
project/
  ├── @types/
  ├── config/
  ├── logs/
  ├── public/
  ├── src/
  |    ├── api/
  |    |    ├── user/
  |    |    |   ├── controller/
  |    |    |   ├── entity/
  |    |    |   ├── repository/
  |    |    |   └── service/
  |    |    ├── post/
  |    |    └── …/
  |    ├── routes/
  |    ├── middlewares/
  |    ├── utils/
  |    └── app.ts (또는 server.ts)
  ├── views/
  └── .env
```

##### REST API 서비스
- 3 계층 구조
  - Controller
  - Service
  - Data
- 백그라운드 작업
  - 이벤트 발생
- 의존성 주입
  - `express.js` 등
- 비밀번호 · secrets · API key
  - 누출 X
  - configuration manager 사용
- node.js 서버 설정 파일
  - 작은 모듈들 분리 <sub>(독립적 로드)</sub>
```
src
├── app.js      # App entry point
├── api         # Express route controllers for all the endpoints of the app
├── config      # Environment variables and configuration related stuff
├── jobs        # Jobs definitions for agenda.js
├── loaders     # Split the startup process into modules
├── models      # Database models
├── services    # All the business logic is here
├── subscribers # Event handlers for async task
└── types       # Type declaration files (d.ts) for Typescript
```

![3_Layer_Architecture_1](./images/3_Layer_Architecture_1.jfif)

![3_Layer_Architecture_2](./images/3_Layer_Architecture_2.jfif)
