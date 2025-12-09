# React Native Testing Demo.

Demo project hướng dẫn **Unit Testing** với **Jest** và **React Native Testing Library** cho môn **Phát triển Ứng dụng trên Thiết bị Di động (CO3043)**.

[![Run Tests](https://github.com/MobileApp251/demo_mobile/actions/workflows/test.yml/badge.svg)](https://github.com/MobileApp251/demo_mobile/actions/workflows/test.yml)
[![SonarCloud](https://github.com/MobileApp251/demo_mobile/actions/workflows/sonarcloud.yml/badge.svg)](https://github.com/MobileApp251/demo_mobile/actions/workflows/sonarcloud.yml)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=MobileApp251_react-test&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=MobileApp251_react-test)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=MobileApp251_react-test&metric=coverage)](https://sonarcloud.io/summary/new_code?id=MobileApp251_react-test)

## Tổng quan

Project này demo các kỹ thuật testing cho React Native app bao gồm:

- ✅ **Unit Testing** với Jest + React Native Testing Library
- 🤖 **Tự động hóa CI/CD** với GitHub Actions
- 📊 **Phân tích chất lượng** với SonarCloud
- 📈 **Coverage tracking** và reporting

## Screenshots

### Onboarding Screen

Màn hình giới thiệu với 3 slides, navigation và pagination.

### Home Screen

Task management app với add, toggle, và delete tasks.

## Cấu trúc project

```
demo_mobile/
├── src/
│   └── screens/
│       ├── OnboardingScreen/
│       │   ├── OnboardingScreen.tsx
│       │   └── OnboardingScreen.test.tsx
│       └── HomeScreen/
│           ├── HomeScreen.tsx
│           └── HomeScreen.test.tsx
├── .github/
│   └── workflows/
│       ├── test.yml              # GitHub Actions workflow cho testing
│       └── sonarcloud.yml        # GitHub Actions workflow cho SonarCloud
├── App.tsx
├── App.test.tsx
├── jest.setup.js                 # Jest configuration
├── sonar-project.properties      # SonarCloud configuration
└── package.json
```

## Test Coverage

Hiện tại project đạt **97.72% coverage**:

| File                 | % Stmts | % Branch | % Funcs | % Lines |
| -------------------- | ------- | -------- | ------- | ------- |
| **All files**        | 97.72   | 90       | 94.44   | 97.61   |
| App.tsx              | 80      | 50       | 50      | 80      |
| HomeScreen.tsx       | 100     | 100      | 100     | 100     |
| OnboardingScreen.tsx | 100     | 87.5     | 100     | 100     |

---

## Phần 3: Unit Test với Jest + React Native Testing Library

### Bước 1: Khởi tạo dự án Expo

```bash
# Tạo project mới
npx create-expo-app@latest demo_mobile --template blank-typescript

# Di chuyển vào thư mục project
cd demo_mobile
```

### Bước 2: Cài đặt Testing dependencies

```bash
npm install --save-dev --legacy-peer-deps \
  jest \
  @testing-library/react-native \
  @testing-library/jest-native \
  jest-expo \
  @types/jest \
  react-test-renderer@19.1.0
```

**Lưu ý**: Sử dụng `--legacy-peer-deps` để tránh conflict dependencies với React 19.

### Bước 3: Cấu hình Jest

Thêm vào `package.json`:

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "jest": {
    "preset": "jest-expo",
    "setupFilesAfterEnv": ["<rootDir>/jest.setup.js"],
    "testEnvironment": "node",
    "transformIgnorePatterns": [
      "node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@unimodules/.*|unimodules|sentry-expo|native-base|react-native-svg)"
    ],
    "collectCoverageFrom": [
      "**/*.{ts,tsx}",
      "!**/coverage/**",
      "!**/node_modules/**",
      "!**/babel.config.js",
      "!**/jest.setup.js",
      "!**/*.test.{ts,tsx}",
      "!**/index.ts"
    ],
    "coverageReporters": ["json-summary", "text", "lcov", "html"]
  }
}
```

Tạo file `jest.setup.js`:

```javascript
global.setImmediate =
  global.setImmediate || ((fn, ...args) => global.setTimeout(fn, 0, ...args));

global.__ExpoImportMetaRegistry = {
  register: () => {},
  get: () => null,
};

global.structuredClone =
  global.structuredClone || ((obj) => JSON.parse(JSON.stringify(obj)));
```

### Bước 4: Viết Component và Tests

#### OnboardingScreen.tsx

```typescript
// src/screens/OnboardingScreen/OnboardingScreen.tsx
import React, { useState } from "react";
import { View, Text, TouchableOpacity, StyleSheet } from "react-native";

// ... (xem full code trong file)
```

#### OnboardingScreen.test.tsx

```typescript
// src/screens/OnboardingScreen/OnboardingScreen.test.tsx
import React from "react";
import { render, fireEvent, screen } from "@testing-library/react-native";
import OnboardingScreen from "./OnboardingScreen";

describe("OnboardingScreen", () => {
  it("should render without crashing", () => {
    render(<OnboardingScreen />);
    expect(screen.getByTestId("onboarding-screen")).toBeTruthy();
  });

  it("should navigate to next slide when Next button is pressed", () => {
    render(<OnboardingScreen />);
    const nextButton = screen.getByTestId("next-button");
    fireEvent.press(nextButton);
    expect(screen.getByTestId("onboarding-title")).toHaveTextContent(
      "Test with Confidence"
    );
  });

  // ... thêm các test cases khác
});
```

### Bước 5: Chạy tests

```bash
# Chạy tất cả tests
npm test

# Chạy tests với watch mode
npm run test:watch

# Chạy tests với coverage
npm run test:coverage
```

#### Kết quả mong đợi:

```
Test Suites: 3 passed, 3 total
Tests:       40 passed, 40 total
Coverage:    97.72%
```

---

## Phần 4: Tự động hóa Test với GitHub Actions

### Bước 1: Tạo GitHub repository

```bash
git init
git add .
git commit -m "Initial commit with tests"
git branch -M main
git remote add origin https://github.com/MobileApp251/demo_mobile.git
git push -u origin main
```

### Bước 2: Tạo GitHub Actions workflow

Tạo file `.github/workflows/test.yml`:

```yaml
name: Run Tests

on:
  push:
    branches: [master, main, develop]
  pull_request:
    branches: [master, main, develop]

jobs:
  test:
    name: Unit Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci --legacy-peer-deps

      - name: Run tests
        run: npm test -- --ci --coverage --maxWorkers=2

      - name: Upload coverage reports
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
```

### Bước 3: Xem kết quả

1. Push code lên GitHub
2. Vào tab **Actions** trong GitHub repository
3. Xem workflow **Run Tests** đang chạy
4. Kiểm tra **Artifacts** để download coverage report

### Bước 4: Thêm badge vào README

```markdown
[![Run Tests](https://github.com/MobileApp251/demo_mobile/actions/workflows/test.yml/badge.svg)](https://github.com/MobileApp251/demo_mobile/actions/workflows/test.yml)
```

---

## Phần 5: Phân tích chất lượng với SonarCloud

### Bước 1: Tạo account SonarCloud

1. Truy cập [sonarcloud.io](https://sonarcloud.io)
2. Đăng nhập bằng GitHub account
3. Click **Analyze new project**
4. Chọn repository `demo_mobile`

### Bước 2: Lấy thông tin project

Sau khi tạo project, lấy thông tin:

- **Organization Key**: Tìm trong Settings → Organization
- **Project Key**: Hiển thị trên dashboard

### Bước 3: Cấu hình SonarCloud

Tạo file `sonar-project.properties`:

```properties
sonar.projectKey=MobileApp251_react-test
sonar.organization=YOUR_ORGANIZATION_KEY

sonar.projectName=Demo Mobile - React Native Testing
sonar.projectVersion=1.0

sonar.sources=src,App.tsx
sonar.tests=src
sonar.test.inclusions=**/*.test.tsx,**/*.test.ts
sonar.exclusions=**/node_modules/**,**/coverage/**,**/*.test.tsx,**/*.test.ts

sonar.typescript.lcov.reportPaths=coverage/lcov.info
sonar.javascript.lcov.reportPaths=coverage/lcov.info
```

### Bước 4: Thêm SONAR_TOKEN vào GitHub Secrets

1. Vào SonarCloud → **My Account** → **Security** → **Generate Token**
2. Copy token
3. Vào GitHub repo → **Settings** → **Secrets and variables** → **Actions**
4. Click **New repository secret**
5. Name: `SONAR_TOKEN`, Value: paste token

### Bước 5: Tạo SonarCloud workflow

Tạo file `.github/workflows/sonarcloud.yml`:

```yaml
name: SonarCloud Analysis

on:
  push:
    branches: [master, main, develop]
  pull_request:
    branches: [master, main, develop]

jobs:
  sonarcloud:
    name: SonarCloud
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci --legacy-peer-deps

      - name: Run tests with coverage
        run: npm run test:coverage

      - name: SonarCloud Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### Bước 6: Xem kết quả trên SonarCloud

1. Push code với cấu hình mới
2. Đợi workflow chạy xong
3. Vào [sonarcloud.io](https://sonarcloud.io) → chọn project
4. Xem các metrics:
   - **Coverage**: % code được test
   - **Reliability**: Bugs
   - **Security**: Vulnerabilities
   - **Maintainability**: Code smells
   - **Duplications**: Code trùng lặp

### Bước 7: Thêm badges vào README

```markdown
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=MobileApp251_react-test&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=MobileApp251_react-test)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=MobileApp251_react-test&metric=coverage)](https://sonarcloud.io/summary/new_code?id=MobileApp251_react-test)
```

---

## Best Practices

### 1. Viết test song song với code

```bash
# Mỗi component mới → tạo file test ngay
ComponentName.tsx
ComponentName.test.tsx
```

### 2. Chạy test locally trước khi push

```bash
npm run test:watch
```

### 3. Sử dụng testID cho UI testing

```typescript
<View testID="my-component">
  <Text testID="title">Hello</Text>
</View>
```

### 4. Test các trường hợp edge cases

- Empty state
- Loading state
- Error state
- User interactions

### 5. Maintain high coverage (≥ 70%)

```bash
npm run test:coverage
```

---

## Các lỗi thường gặp và cách khắc phục

### 1. Lỗi: "Incorrect version of react-test-renderer"

**Giải pháp:**

```bash
npm install -D react-test-renderer@19.1.0 --legacy-peer-deps
```

### 2. Lỗi: "You are trying to import a file outside of the scope"

**Giải pháp:** Thêm vào `jest.setup.js`:

```javascript
global.__ExpoImportMetaRegistry = {
  register: () => {},
  get: () => null,
};
```

### 3. Lỗi: GitHub Actions failed

**Giải pháp:**

- Kiểm tra `npm ci --legacy-peer-deps` trong workflow
- Xem logs chi tiết trong Actions tab

### 4. SonarCloud không nhận coverage

**Giải pháp:**

- Kiểm tra file `sonar-project.properties`
- Đảm bảo `coverage/lcov.info` được generate
- Verify SONAR_TOKEN đã được thêm vào GitHub Secrets

---

## Nội dung bài tập trên lớp

Mỗi nhóm nộp:

### 1. Link SonarCloud project

- Coverage ≥ 70%
- Maintainability Rating ≥ A
- Reliability Rating ≥ A
- Security Rating ≥ A

### 2. Screenshots

- SonarCloud Dashboard
- OnboardingScreen tests (≥ 3 test cases)
- HomeScreen tests (≥ 3 test cases)
- GitHub Actions workflow success

### 3. README.md

- Hướng dẫn chạy test locally
- Badges (Tests, SonarCloud, Coverage)

---

## Tài liệu tham khảo

- [Jest Documentation](https://jestjs.io/)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [GitHub Actions](https://docs.github.com/en/actions)
- [SonarCloud Documentation](https://docs.sonarcloud.io/)

---

## License

MIT License - Dự án demo cho mục đích học tập.

## Contributors

- **Nhóm [L01_Cutephomaique]** - CO3043 - Học kỳ [HK]

---

**Lưu ý**: Thay thế `MobileApp251`, `MobileApp251_react-test`, `YOUR_ORGANIZATION_KEY` bằng thông tin thực tế của bạn.
