# 🍜 PWD Week 4 - Express & Render로 만드는 Ajou Campus Foodmap API

> **실전 웹서비스 개발 4주차**: Node.js와 Express로 백엔드 REST API를 구현하고, Render를 통해 3주차 React 클라이언트와 연동 가능한 서버를 배포.
## 📚 주차 자료
- [4주차 강의자료](https://drive.google.com/file/d/1R_7M_Ub0X-gq4iD2GPaTVBmDK39UVFW6/view?usp=drive_link)

---

## 🛠️ 프로젝트 소개

### 만들어볼 서비스
**Ajou Campus Foodmap API** - 아주대학교 주변 맛집 데이터를 제공하고 관리할 수 있는 백엔드 서비스.

### 주요 기능
- `/health`: 서버 상태 확인용 헬스 체크
- `/api/restaurants`: 맛집 목록 비동기 조회 (지연 시뮬레이션 포함)
- `/api/restaurants/sync-demo`: 동기 방식 목록 조회 비교 엔드포인트
- `/api/restaurants/:id`: 개별 맛집 상세 조회
- `/api/restaurants/popular?limit=`: 평점 상위 맛집 조회
- `POST /api/restaurants`: 추천 메뉴를 배열로 정규화하며 신규 맛집 등록
- `POST /api/restaurants/reset-demo`: 데모 데이터 초기화

### 사용 기술
- **Runtime**: Node.js 22
- **Framework**: Express 5, cors 미들웨어
- **구조**: MVCS (controllers / services / routes / middleware)
- **테스트**: `npm test`
- **배포**: Render Web Service (Build Command `npm install`, Start Command `npm start`)

---

## ✅ 사전 준비사항

### 1. 필수 프로그램 설치 확인
```bash
node --version    # 22.0.0 이상 권장
npm --version     # 10.x 이상
git --version
```
> Node.js가 없다면 [nodejs.org](https://nodejs.org) LTS 버전을, Git은 [git-scm.com](https://git-scm.com)에서 설치하세요.

### 2. 계정 준비
- GitHub 계정 (코드 저장 및 협업)
- Render 계정 ([render.com](https://render.com), GitHub 연동 권장)

### 3. 개발 도구
- VS Code (또는 선호 IDE)
- 추천 확장: ESLint, Prettier, REST Client 혹은 Thunder Client
- API 테스트 도구: Postman, 혹은 단순히 `curl`

---

## 🚀 프로젝트 시작하기

### Step 1: Express 프로젝트 개발환경 구축

```bash
# Express 프로젝트 폴더 생성
mkdir pwd-week4

# 프로젝트 폴더로 이동
cd pwd-week4

# 프로젝트 생성
$ npm install express --save
```

### Step 2: 프로젝트 GitHub 저장소 연동
// 깃허브 접속 후 pwd-week3 저장소 생성 후

```bash
# 현재 폴더를 Git 저장소(Repository)로 초기화(Initialize)
git init

# 모든 변경 파일을 스테이징(Staging Area)에 추가
git add .

# 스테이징된 파일을 커밋(Commit)으로 기록 (-m: 메시지(Message) 옵션)
git commit -m "Init pwd-week4 : Express"

# 기본 브랜치(Branch) 이름을 main으로 변경 (-m: move)
git branch -m main

# 원격(Remote) 저장소 'origin' 등록 (GitHub URL로 연결)
git remote add origin https://github.com/<username>/pwd-week4.git

# 로컬 main을 원격 origin/main으로 최초 푸시(Push) (-u: 업스트림(Upstream) 설정)
#  → 이후에는 git push 만으로도 동일 브랜치에 푸시 가능
git push -u origin main
```

// 이후 코드 수정 시
```bash
git add .

git commit -m "....."

git push -u origin main
```

### Step 3. 의존성 설치 & 로컬 서버 실행
패키지 의존성(package.json) 설정
```json
{
  "name": "pwd-week4",
  "version": "1.0.0",
  "private": true,
  "main": "server.js",
  "scripts": {
    "dev": "nodemon server.js",
    "start": "node server.js",
    "test": "jest --runInBand",
    "test:watch": "jest --watch --runInBand"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^5.1.0"
  },
  "devDependencies": {
    "jest": "^29.7.0",
    "nodemon": "^3.1.7",
    "supertest": "^7.0.0"
  }
}
```

```bash
# 패키지 설치
npm install

# 개발용 서버 실행
npm run dev
```

- 웹 브라우저 기본 주소: `http://localhost:3000`
- 응답 메시지 확인

```bash
# 헬스 체크
curl http://localhost:3000/health
```

### Step 4. 폴더 구조 이해
```
src/
 ├─ app.js                # Express 앱 구성
 ├─ routes/               # 라우터 정의
 ├─ controllers/          # 요청/응답 컨트롤러
 ├─ services/             # 비즈니스 로직 (동기/비동기 비교)
 ├─ middleware/           # NotFound, Error 핸들러
 ├─ data/restaurants.json # 시드 데이터
 └─ utils/asyncHandler.js # 공통 비동기 에러 래퍼
tests/                    # 모듈 테스트 코드
server.js                 # Node Entry Point
```

### Step 5. 프로젝트 기능 구현

```js
// server.js
const createApp = require('./src/app');

const PORT = process.env.PORT || 3000;
const app = createApp();

if (require.main === module) {
  app.listen(PORT, () => {
    console.log(`Server listening on port ${PORT}`);
  });
}

module.exports = app;
```

```js
// src/app.js
const express = require('express');
const cors = require('cors');
const restaurantsRouter = require('./routes/restaurants.routes');
const notFound = require('./middleware/notFound.middleware');
const errorHandler = require('./middleware/error.middleware');

function createApp() {
  const app = express();

  app.use(cors());
  app.use(express.json());
  app.use(express.urlencoded({ extended: true }));

  app.get('/health', (req, res) => {
    res.json({ status: 'ok' });
  });

  app.use('/api/restaurants', restaurantsRouter);

  app.use(notFound);
  app.use(errorHandler);

  return app;
}

module.exports = createApp;
```

- src/data/restaurants.json
```json
[
  {
    "id": 1,
    "name": "송림식당",
    "category": "한식",
    "location": "경기 수원시 영통구 월드컵로193번길 21 원천동",
    "priceRange": "7,000-13,000원",
    "rating": 4.99,
    "description": "최고의 한식 맛집입니다.",
    "recommendedMenu": ["순두부", "김치찌개","소불고기", "제육볶음"],
    "likes": 0,
    "image": "https://mblogthumb-phinf.pstatic.net/MjAyMjA2MTJfODEg/MDAxNjU0OTYzNTM3MjE1.1BfmrmOsz_B6DBHAnhQSs6qfNIDnssofR-DrzMfigIIg.JHHDheG6ifJjtfKUqLss_mLXWFE9fNJ5BmepNUVXSOog.PNG.cary63/image.png?type=w966"
  },
  {
    "id": 2,
    "name": "별미떡볶이",
    "category": "분식",
    "location": "경기 수원시 영통구 아주로 42 아카데미빌딩",
    "priceRange": "7,000-10,000원",
    "rating": 4.92,
    "description": "바삭한 튀김과 함께하는 행복한 한입",
    "recommendedMenu": ["떡볶이", "튀김", "순대", "어묵"],
    "likes": 0,
    "image": "https://search.pstatic.net/common/?src=http%3A%2F%2Fblogfiles.naver.net%2FMjAyNTA4MTJfMjcg%2FMDAxNzU0OTQ5ODk1Mjg0.GR6i3mNpJJXyqQrozGEJ65InCDBGlEmxc0aCeVHncJgg.sduDPX67J8hhoGxq4vLohpS4dXk1w-706dQLPfVs1iwg.JPEG%2Foutput%25A3%25DF1564208956.jpg"
  },
  {
    "id": 3,
    "name": "SOGO",
    "category": "일식",
    "location": "경기 수원시 영통구 월드컵로193번길 7",
    "priceRange": "10,000-16,000원",
    "rating": 4.89,
    "description": "일식 맛집, 구 허수아비,",
    "recommendedMenu": ["냉모밀", "김치돈까스나베", "코돈부르"],
    "likes": 0,
    "image": "https://search.pstatic.net/common/?src=https%3A%2F%2Fldb-phinf.pstatic.net%2F20190707_63%2F1562462598960nPDMy_JPEG%2FW7iKQEhTMzCF3flC1t0pzgzF.jpeg.jpg"
  }
]
```

```js
// src/routes/restaurants.routes.js
const express = require('express');
const restaurantsController = require('../controllers/restaurants.controller');

const router = express.Router();

router.get('/', restaurantsController.getRestaurants);
router.get('/sync-demo', restaurantsController.getRestaurantsSync);
router.get('/popular', restaurantsController.getPopularRestaurants);
router.get('/:id', restaurantsController.getRestaurant);
router.post('/', restaurantsController.createRestaurant);
router.post('/reset-demo', restaurantsController.resetDemoData);

module.exports = router;
```

```js
// src/middleware/error.middleware.js
module.exports = function errorHandler(err, req, res, next) {
  const status = err.statusCode || 500;
  const payload = {
    error: {
      message: err.message || 'Internal Server Error'
    }
  };

  if (process.env.NODE_ENV !== 'production' && err.stack) {
    payload.error.stack = err.stack;
  }

  res.status(status).json(payload);
};
```

```js
// src/middleware/notFound.middleware.js
module.exports = function notFound(req, res) {
  res.status(404).json({ error: { message: 'Resource not found' } });
};
```

```js
// src/controllers/restaurants.controller.js
const restaurantService = require('../services/restaurants.service');
const asyncHandler = require('../utils/asyncHandler');

const normaliseMenu = (menu) => {
  if (!menu) return [];
  if (Array.isArray(menu)) return menu;
  if (typeof menu === 'string') {
    return menu
      .split(',')
      .map((item) => item.trim())
      .filter(Boolean);
  }
  return [];
};

exports.getRestaurants = asyncHandler(async (req, res) => {
  const restaurants = await restaurantService.getAllRestaurants();
  res.json({ data: restaurants });
});

exports.getRestaurantsSync = (req, res) => {
  const restaurants = restaurantService.getAllRestaurantsSync();
  res.json({
    data: restaurants,
    meta: {
      execution: 'synchronous'
    }
  });
};

exports.getRestaurant = asyncHandler(async (req, res) => {
  const restaurant = await restaurantService.getRestaurantById(req.params.id);

  if (!restaurant) {
    res.status(404).json({ error: { message: 'Restaurant not found' } });
    return;
  }

  res.json({ data: restaurant });
});

exports.getPopularRestaurants = asyncHandler(async (req, res) => {
  const limit = req.query.limit ? Number(req.query.limit) : 5;
  const restaurants = await restaurantService.getPopularRestaurants(limit);
  res.json({ data: restaurants });
});

exports.createRestaurant = asyncHandler(async (req, res) => {
  const payload = {
    ...req.body,
    recommendedMenu: normaliseMenu(req.body?.recommendedMenu)
  };

  const restaurant = await restaurantService.createRestaurant(payload);
  res.status(201).json({ data: restaurant });
});

exports.resetDemoData = asyncHandler(async (req, res) => {
  restaurantService.resetStore();
  const restaurants = await restaurantService.getAllRestaurants();
  res.json({ data: restaurants });
});
```

```js
// src/services/restaurants.service.js
const path = require('path');
const { readFile } = require('fs/promises');
const { readFileSync } = require('fs');
const Restaurant = require('../models/restaurant.model');

const DATA_PATH = path.join(__dirname, '..', 'data', 'restaurants.json');
let restaurantCache = loadRestaurantsSync();

function loadRestaurantsSync() {
  const raw = readFileSync(DATA_PATH, 'utf8');
  const parsed = JSON.parse(raw);
  return parsed.map((item) => new Restaurant(item));
}

async function loadRestaurantsAsync() {
  const raw = await readFile(DATA_PATH, 'utf8');
  const parsed = JSON.parse(raw);
  return parsed.map((item) => new Restaurant(item));
}

async function ensureCache() {
  if (!restaurantCache || restaurantCache.length === 0) {
    restaurantCache = await loadRestaurantsAsync();
  }
  return restaurantCache;
}

function nextRestaurantId() {
  return restaurantCache.reduce((max, restaurant) => Math.max(max, restaurant.id), 0) + 1;
}

function cloneCollection(collection) {
  return collection.map((restaurant) => new Restaurant(restaurant));
}

// Simulates non-blocking I/O latency so students can compare async vs sync flows.
async function simulateLatency(delayInMs = 20) {
  return new Promise((resolve) => setTimeout(resolve, delayInMs));
}

async function getAllRestaurants() {
  await ensureCache();
  await simulateLatency();
  return cloneCollection(restaurantCache);
}

function getAllRestaurantsSync() {
  if (!restaurantCache || restaurantCache.length === 0) {
    restaurantCache = loadRestaurantsSync();
  }
  return cloneCollection(restaurantCache);
}

async function getRestaurantById(id) {
  await ensureCache();
  await simulateLatency();
  const numericId = Number(id);
  return restaurantCache.find((restaurant) => restaurant.id === numericId) || null;
}

async function getPopularRestaurants(limit = 5) {
  const restaurants = await getAllRestaurants();
  return restaurants
    .sort((a, b) => b.rating - a.rating)
    .slice(0, limit);
}

async function createRestaurant(payload) {
  await ensureCache();
  await simulateLatency();

  const requiredFields = ['name', 'category', 'location'];
  const missingField = requiredFields.find((field) => !payload[field]);

  if (missingField) {
    const error = new Error(`'${missingField}' is required`);
    error.statusCode = 400;
    throw error;
  }

  const restaurant = new Restaurant({
    id: nextRestaurantId(),
    name: payload.name,
    category: payload.category,
    location: payload.location,
    priceRange: payload.priceRange ?? '정보 없음',
    rating: payload.rating ?? 0,
    description: payload.description ?? '',
    recommendedMenu: payload.recommendedMenu ?? [],
    likes: 0,
    image: payload.image ?? ''
  });

  restaurantCache = [...restaurantCache, restaurant];
  return new Restaurant(restaurant);
}

function resetStore() {
  restaurantCache = loadRestaurantsSync();
}

module.exports = {
  getAllRestaurants,
  getAllRestaurantsSync,
  getRestaurantById,
  getPopularRestaurants,
  createRestaurant,
  resetStore,
  loadRestaurantsSync,
  loadRestaurantsAsync,
};
```

```js
// src/models/restaurant.model.js
class Restaurant {
  constructor({
    id,
    name,
    category,
    location,
    priceRange,
    rating,
    description,
    recommendedMenu = [],
    likes = 0,
    image = ''
  }) {
    this.id = Number(id);
    this.name = name;
    this.category = category;
    this.location = location;
    this.priceRange = priceRange;
    this.rating = Number(rating);
    this.description = description;
    this.recommendedMenu = [...recommendedMenu];
    this.likes = Number(likes);
    this.image = image;
  }

  updateLikes(likes) {
    this.likes = Number(likes);
    return this;
  }
}

module.exports = Restaurant;
```

```js
// src/utils/asyncHandler.js
module.exports = (handler) => (req, res, next) => {
  Promise.resolve(handler(req, res, next)).catch(next);
};
```

- 테스트 코드 작성성
```js
// tests/restaurants.routes.test.js
const request = require('supertest');
const createApp = require('../src/app');
const restaurantService = require('../src/services/restaurants.service');

describe('Restaurant routes', () => {
  let app;

  beforeEach(() => {
    restaurantService.resetStore();
    app = createApp();
  });

  test('GET /api/restaurants returns a list', async () => {
    const response = await request(app).get('/api/restaurants');
    expect(response.status).toBe(200);
    expect(response.body.data).toBeInstanceOf(Array);
  });

  test('GET /api/restaurants/sync-demo flags synchronous execution', async () => {
    const response = await request(app).get('/api/restaurants/sync-demo');
    expect(response.status).toBe(200);
    expect(response.body.meta.execution).toBe('synchronous');
  });

  test('GET /api/restaurants/:id returns an item', async () => {
    const response = await request(app).get('/api/restaurants/1');
    expect(response.status).toBe(200);
    expect(response.body.data.id).toBe(1);
  });

  test('GET /api/restaurants/:id handles missing items', async () => {
    const response = await request(app).get('/api/restaurants/999');
    expect(response.status).toBe(404);
    expect(response.body.error.message).toContain('not found');
  });

  test('POST /api/restaurants validates payload', async () => {
    const response = await request(app)
      .post('/api/restaurants')
      .send({ name: '테스트' })
      .set('Content-Type', 'application/json');

    expect(response.status).toBe(400);
    expect(response.body.error.message).toContain('category');
  });

  test('POST /api/restaurants creates a restaurant', async () => {
    const payload = {
      name: '새로운 식당',
      category: '카페',
      location: '캠퍼스 타운',
    };

    const response = await request(app)
      .post('/api/restaurants')
      .send(payload)
      .set('Content-Type', 'application/json');

    expect(response.status).toBe(201);
    expect(response.body.data.name).toBe(payload.name);
  });
});
```

```js
// tests/restaurants.service.test.js
const restaurantService = require('../src/services/restaurants.service');

describe('RestaurantService', () => {
  afterEach(() => {
    restaurantService.resetStore();
  });

  test('getAllRestaurants resolves with data', async () => {
    const restaurants = await restaurantService.getAllRestaurants();
    expect(Array.isArray(restaurants)).toBe(true);
    expect(restaurants.length).toBeGreaterThan(0);
  });

  test('getAllRestaurantsSync returns data immediately', () => {
    const restaurants = restaurantService.getAllRestaurantsSync();
    expect(Array.isArray(restaurants)).toBe(true);
    expect(restaurants.length).toBeGreaterThan(0);
  });

  test('createRestaurant appends a new entry', async () => {
    const payload = {
      name: '테스트 식당',
      category: '테스트',
      location: '가상 캠퍼스',
      rating: 4.5,
    };

    const created = await restaurantService.createRestaurant(payload);
    expect(created.id).toBeDefined();
    expect(created.name).toBe(payload.name);

    const all = await restaurantService.getAllRestaurants();
    const found = all.find((item) => item.id === created.id);
    expect(found).toBeTruthy();
  });

  test('createRestaurant rejects invalid payloads', async () => {
    await expect(
      restaurantService.createRestaurant({ name: '누락된 식당' })
    ).rejects.toThrow("'category' is required");
  });
});

```


### Step 6. API 응답 확인
```bash
# 전체 맛집 목록
curl http://localhost:3000/api/restaurants

# 인기 맛집 3개만 보기
curl "http://localhost:3000/api/restaurants/popular?limit=3"

# 신규 맛집 추가 (PowerShell)
curl -X POST http://localhost:3000/api/restaurants ^
  -H "Content-Type: application/json" ^
  -d "{\"name\":\"Ajou Cafeteria\",\"location\":\"기숙사 1층\",\"recommendedMenu\":[\"라면\",\"김밥\"],\"rating\":4.6}"
```
> macOS/Linux에서는 줄바꿈 기호로 `\` 를 사용하세요.

### Step 7. 테스트 실행
```bash
npm test
```
- 서비스 로직 + REST 엔드포인트 통합 테스트가 통과해야 합니다.
- 실패한다면 서버가 실행 중인지(`npm run dev`), 필요 시 `POST /api/restaurants/reset-demo` 로 초기화했는지 확인하세요.

---

### Step 8. Render 배포하기
1. [render.com](https://render.com) 접속 → `Sign In with GitHub` 선택.
2. Render가 GitHub 리포지토리에 접근할 수 있도록 권한을 허용합니다.
3. 대시보드에서 `New +` → `Web Service` 를 선택합니다.

### Step 9. 배포 설정
1. 연결할 저장소로 `pwd-week4` 를 선택합니다. (보이지 않는다면 `Configure account`에서 리포지토리를 허용하세요.)
2. 환경 설정을 다음과 같이 입력합니다.
   - **Name**: `pwd-week4-<nickname>`
   - **Region**: Singapore (ap-southeast-1) 권장
   - **Branch**: `main`
   - **Root Directory**: 비워둠
   - **Build Command**: `npm install`
   - **Start Command**: `npm start`
   - **Instance Type**: Free
   - **Auto Deploy**: Yes
3. (선택) Environment Variables 추가
   - 향후 비밀 키나 DB URL 등도 이곳에서 관리할 예정
4. `Create Web Service` 클릭 → 빌드 로그가 성공(`Live`)인지 확인합니다.

### Step 10. 배포 검증
```bash
# Health 체크
curl https://pwd-week4-<username>.onrender.com/health

# API 목록 조회
curl https://pwd-week4-<username>.onrender.com/api/restaurants

# 데모 데이터 초기화
curl -X POST https://pwd-week4-<username>.onrender.com/api/restaurants/popular
```
- Render Free 플랜은 유휴 상태에서 잠든 뒤 첫 호출이 느릴 수 있습니다.
- 3주차 React 앱에서 API를 호출하려면 `.env` 또는 환경 설정에 Render 도메인을 등록하세요.

---

### Step 10. pwd-week3 src/services/api.jsx 수정 후 Netlify 재 배포
```jsx
import axios from 'axios';

const fallbackRestaurants = [
  {
    id: 1,
    name: 'Songlim Restaurant',
    category: 'Korean food',
    location: 'Gyeonggi-do Suwon-si Yeongtong-gu World Cup-ro 193beon-gil 21 (Namcheon-dong)',
    priceRange: 'KRW 7,000-13,000',
    rating: 4.99,
    description: 'Beloved spot for comforting Korean dishes near campus.',
    recommendedMenu: [
      'Soft tofu stew',
      'Kimchi stew',
      'Bulgogi',
      'Spicy pork stir-fry'
    ],
    likes: 0,
    image: 'https://mblogthumb-phinf.pstatic.net/MjAyMjA2MTJfODEg/MDAxNjU0OTYzNTM3MjE1.1BfmrmOsz_B6DBHAnhQSs6qfNIDnssofR-DrzMfigIIg.JHHDheG6ifJjtfKUqLss_mLXWFE9fNJ5BmepNUVXSOog.PNG.cary63/image.png?type=w966'
  },
  {
    id: 2,
    name: 'Special Tteokbokki',
    category: 'Snacks',
    location: '42 Gwanak-ro, Yeongtong-gu, Suwon-si',
    priceRange: 'KRW 7,000-10,000',
    rating: 4.92,
    description: 'Crispy fritters and spicy tteokbokki that locals love.',
    recommendedMenu: [
      'Tteokbokki',
      'Fried seaweed roll',
      'Fish cake soup',
      'Fried dumplings'
    ],
    likes: 0,
    image: 'https://search.pstatic.net/common/?src=http%3A%2F%2Fblogfiles.naver.net%2FMjAyNTA4MTJfMjcg%2FMDAxNzU0OTQ5ODk1Mjg0.GR6i3mNpJJXyqQrozGEJ65InCDBGlEmxc0aCeVHncJgg.sduDPX67J8hhoGxq4vLohpS4dXk1w-706dQLPfVs1iwg.JPEG%2Foutput%25A3%25DF1564208956.jpg'
  },
  {
    id: 3,
    name: 'SOGO',
    category: 'Japanese food',
    location: '7 World Cup-ro 193beon-gil, Yeongtong-gu, Suwon-si',
    priceRange: 'KRW 10,000-16,000',
    rating: 4.89,
    description: 'Casual Japanese restaurant with generous portions.',
    recommendedMenu: [
      'Cold soba set',
      'Kimchi pork cutlet rice bowl',
      'Cordon bleu'
    ],
    likes: 0,
    image: 'https://search.pstatic.net/common/?src=https%3A%2F%2Fldb-phinf.pstatic.net%2F20190707_63%2F1562462598960nPDMy_JPEG%2FW7iKQEhTMzCF3flC1t0pzgzF.jpeg.jpg'
  }
];

const DEFAULT_BASE_URL = 'https://pwd-week4-<username>.onrender.com';
const rawBaseUrl = import.meta.env?.VITE_API_BASE_URL || DEFAULT_BASE_URL;
const API_BASE_URL = rawBaseUrl.endsWith('/') ? rawBaseUrl.slice(0, -1) : rawBaseUrl;

const api = axios.create({
  baseURL: API_BASE_URL,
  timeout: 10000,
});

api.interceptors.request.use(
  (config) => {
    console.log('API request:', config.method?.toUpperCase(), config.url);
    return config;
  },
  (error) => Promise.reject(error)
);

api.interceptors.response.use(
  (response) => response,
  (error) => {
    console.error('API error:', error);
    return Promise.reject(error);
  }
);

export const restaurantAPI = {
  getRestaurants: async () => {
    try {
      const response = await api.get('/api/restaurants');
      return response.data;
    } catch (error) {
      console.warn('Using local fallback restaurant list', error);
      return { data: fallbackRestaurants };
    }
  },

  getRestaurantById: async (id) => {
    try {
      const response = await api.get(`/api/restaurants/${id}`);
      return response.data;
    } catch (error) {
      console.warn(`Using local fallback for restaurant ${id}`, error);
      const restaurant = fallbackRestaurants.find(
        (item) => item.id === Number(id)
      );
      return { data: restaurant ?? null };
    }
  },

  getPopularRestaurants: async () => {
    try {
      const response = await api.get('/api/restaurants/popular');
      return response.data;
    } catch (error) {
      console.warn('Using local fallback popular restaurants', error);
      const popular = [...fallbackRestaurants]
        .sort((a, b) => b.rating - a.rating)
        .slice(0, 5);
      return { data: popular };
    }
  },
};

export default api;
```

- 배포 후 Netlify 실행 시 기존과 같이 동작하는지 확인

---

## 📎 제출 가이드

- **제출 마감**: 강의 공지 확인 (예: 10월 5일 23:59)
- **제출 항목**
  - GitHub Repository URL: `https://github.com/<username>/pwd-week4`
  - Render Service URL: `https://pwd-week4-<username>.onrender.com`
  - Netlify Service URL: `https://pwd-week3-<username>.netlify.app`

- **체크리스트**
  - `npm test` 가 로컬에서 통과하나요?
  - 주요 REST 엔드포인트가 Render에서 정상 동작하나요?  
  - React 프론트엔드(3주차)와 연동 테스트를 완료했나요?

---

## 📚 추가 학습 자료
- [Express 공식 문서](https://expressjs.com/ko/)
- [Render - Deploy a Node/Express App](https://docs.render.com/deploy-node-express-app)

---

## 💬 질문 & 도움
- [Practical Web Service TA](https://chatgpt.com/g/g-68bbbf3aa57081919811dd57100b1e46-ajou-digtalmedia-practical-web-service-ta)
- 과제 관련 질문은 Slack `#pwd-week4` 채널 또는 LMS 질의응답 게시판을 활용하세요.

---

## 🎉 수고하셨습니다!
이번 주차를 통해
- Express로 REST API를 설계하고,
- 비동기 처리 & 테스트 코드 작성을 연습하고,
- GitHub ↔ Render 배포 흐름을 경험했습니다.

다음 주차에서는 이 API에 데이터베이스와 인증을 추가해 확장할 예정입니다. 계속해서 화이팅! ✨
