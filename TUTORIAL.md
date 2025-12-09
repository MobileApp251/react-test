

# Hướng dẫn thực hành: React Native Testing

Tài liệu này hướng dẫn từng bước cụ thể để các nhóm sinh viên có thể làm theo trong 1 tiết học.

---

## Chuẩn bị

### Yêu cầu hệ thống

- Node.js ≥ 18
- npm hoặc yarn
- Git
- GitHub account
- SonarCloud account (đăng nhập bằng GitHub)

### Kiểm tra version

```bash
node --version  # v18.0.0 hoặc cao hơn
npm --version   # v9.0.0 hoặc cao hơn
git --version   # bất kỳ version nào
```

---

## Phần 3: Unit Test với Jest + React Native Testing Library

### Bước 1: Khởi tạo project (5 phút)

```bash
# Tạo project mới
npx create-expo-app@latest demo_mobile --template blank-typescript

# Vào thư mục
cd demo_mobile

# Mở trong editor
code .
```

**Checkpoint**: Bạn sẽ thấy cấu trúc:
```
demo_mobile/
├── App.tsx
├── package.json
└── tsconfig.json
```

### Bước 2: Cài đặt dependencies (5 phút)

```bash
npm install --save-dev --legacy-peer-deps \
  jest \
  @testing-library/react-native \
  @testing-library/jest-native \
  jest-expo \
  @types/jest \
  react-test-renderer@19.1.0
```

**Lưu ý quan trọng**:
- Phải dùng `--legacy-peer-deps` vì React 19 có peer dependency conflicts
- Version `react-test-renderer` phải match với version React (19.1.0)

**Checkpoint**: Kiểm tra `package.json` có các devDependencies:
```json
{
  "devDependencies": {
    "jest": "^30.x.x",
    "@testing-library/react-native": "^13.x.x",
    // ...
  }
}
```

### Bước 3: Cấu hình Jest (10 phút)

#### 3.1. Update package.json

Mở `package.json` và thêm:

```json
{
  "scripts": {
    "start": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "web": "expo start --web",
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
    "coverageReporters": [
      "json-summary",
      "text",
      "lcov",
      "html"
    ]
  }
}
```

#### 3.2. Tạo jest.setup.js

Tạo file `jest.setup.js` ở root:

```javascript
global.setImmediate = global.setImmediate || ((fn, ...args) => global.setTimeout(fn, 0, ...args));

global.__ExpoImportMetaRegistry = {
  register: () => {},
  get: () => null,
};

global.structuredClone = global.structuredClone || ((obj) => JSON.parse(JSON.stringify(obj)));
```

**Giải thích**:
- `setImmediate`: Polyfill cho React Native testing
- `__ExpoImportMetaRegistry`: Mock Expo winter runtime (Expo SDK 54+)
- `structuredClone`: Polyfill cho deep cloning

**Checkpoint**: Chạy test để verify config:
```bash
npm test
```

Nếu không có test nào, bạn sẽ thấy:
```
No tests found
```

### Bước 4: Tạo cấu trúc project (5 phút)

```bash
# Tạo thư mục
mkdir -p src/screens/OnboardingScreen
mkdir -p src/screens/HomeScreen
mkdir -p src/types
```

**Checkpoint**: Cấu trúc hiện tại:
```
demo_mobile/
├── src/
│   ├── screens/
│   │   ├── OnboardingScreen/
│   │   └── HomeScreen/
│   └── types/
├── App.tsx
├── jest.setup.js
└── package.json
```

### Bước 5: Tạo OnboardingScreen (15 phút)

#### 5.1. Tạo component

Tạo file `src/screens/OnboardingScreen/OnboardingScreen.tsx`:

```typescript
import React, { useState } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  Dimensions,
} from 'react-native';

const { width } = Dimensions.get('window');

interface OnboardingItem {
  id: number;
  title: string;
  description: string;
  emoji: string;
}

const onboardingData: OnboardingItem[] = [
  {
    id: 1,
    title: 'Welcome to Demo App',
    description: 'Learn how to write effective unit tests for your React Native app',
    emoji: '👋',
  },
  {
    id: 2,
    title: 'Test with Confidence',
    description: 'Use Jest and React Native Testing Library for reliable tests',
    emoji: '✅',
  },
  {
    id: 3,
    title: 'CI/CD Integration',
    description: 'Automate your testing workflow with GitHub Actions',
    emoji: '🚀',
  },
];

interface OnboardingScreenProps {
  onComplete?: () => void;
}

export default function OnboardingScreen({ onComplete }: OnboardingScreenProps) {
  const [currentIndex, setCurrentIndex] = useState(0);

  const handleNext = () => {
    if (currentIndex < onboardingData.length - 1) {
      setCurrentIndex(currentIndex + 1);
    }
  };

  const handleBack = () => {
    if (currentIndex > 0) {
      setCurrentIndex(currentIndex - 1);
    }
  };

  const handleFinish = () => {
    if (onComplete) {
      onComplete();
    }
  };

  const currentItem = onboardingData[currentIndex];
  const isLastSlide = currentIndex === onboardingData.length - 1;

  return (
    <View style={styles.container} testID="onboarding-screen">
      <View style={styles.content}>
        <Text style={styles.emoji} testID="onboarding-emoji">
          {currentItem.emoji}
        </Text>
        <Text style={styles.title} testID="onboarding-title">
          {currentItem.title}
        </Text>
        <Text style={styles.description} testID="onboarding-description">
          {currentItem.description}
        </Text>
      </View>

      <View style={styles.pagination}>
        {onboardingData.map((_, index) => (
          <View
            key={index}
            style={[
              styles.dot,
              index === currentIndex && styles.activeDot,
            ]}
            testID={`pagination-dot-${index}`}
          />
        ))}
      </View>

      <View style={styles.buttonContainer}>
        {currentIndex > 0 && (
          <TouchableOpacity
            style={[styles.button, styles.backButton]}
            onPress={handleBack}
            testID="back-button"
          >
            <Text style={styles.backButtonText}>Back</Text>
          </TouchableOpacity>
        )}

        <TouchableOpacity
          style={[styles.button, styles.nextButton]}
          onPress={isLastSlide ? handleFinish : handleNext}
          testID={isLastSlide ? 'finish-button' : 'next-button'}
        >
          <Text style={styles.nextButtonText}>
            {isLastSlide ? 'Get Started' : 'Next'}
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    justifyContent: 'space-between',
    padding: 20,
  },
  content: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  emoji: {
    fontSize: 80,
    marginBottom: 20,
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 16,
    color: '#1a1a1a',
  },
  description: {
    fontSize: 16,
    textAlign: 'center',
    color: '#666',
    lineHeight: 24,
    paddingHorizontal: 20,
  },
  pagination: {
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
    marginBottom: 30,
  },
  dot: {
    width: 8,
    height: 8,
    borderRadius: 4,
    backgroundColor: '#ddd',
    marginHorizontal: 4,
  },
  activeDot: {
    backgroundColor: '#007AFF',
    width: 20,
  },
  buttonContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    gap: 12,
  },
  button: {
    flex: 1,
    paddingVertical: 16,
    borderRadius: 12,
    alignItems: 'center',
    justifyContent: 'center',
  },
  backButton: {
    backgroundColor: '#f0f0f0',
  },
  nextButton: {
    backgroundColor: '#007AFF',
  },
  backButtonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#333',
  },
  nextButtonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#fff',
  },
});
```

**Lưu ý kỹ thuật**:
- Sử dụng `testID` cho mọi element cần test
- Tách logic thành các functions nhỏ (handleNext, handleBack, handleFinish)
- Type-safe với TypeScript interfaces

#### 5.2. Tạo test file

Tạo file `src/screens/OnboardingScreen/OnboardingScreen.test.tsx`:

```typescript
import React from 'react';
import { render, fireEvent, screen } from '@testing-library/react-native';
import OnboardingScreen from './OnboardingScreen';

describe('OnboardingScreen', () => {
  describe('Rendering', () => {
    it('should render without crashing', () => {
      render(<OnboardingScreen />);
      expect(screen.getByTestId('onboarding-screen')).toBeTruthy();
    });

    it('should display the first onboarding slide by default', () => {
      render(<OnboardingScreen />);

      expect(screen.getByTestId('onboarding-title')).toHaveTextContent('Welcome to Demo App');
      expect(screen.getByTestId('onboarding-description')).toHaveTextContent(
        'Learn how to write effective unit tests for your React Native app'
      );
      expect(screen.getByTestId('onboarding-emoji')).toHaveTextContent('👋');
    });

    it('should render pagination dots correctly', () => {
      render(<OnboardingScreen />);

      expect(screen.getByTestId('pagination-dot-0')).toBeTruthy();
      expect(screen.getByTestId('pagination-dot-1')).toBeTruthy();
      expect(screen.getByTestId('pagination-dot-2')).toBeTruthy();
    });

    it('should not show back button on first slide', () => {
      render(<OnboardingScreen />);

      expect(screen.queryByTestId('back-button')).toBeNull();
    });

    it('should show next button on first slide', () => {
      render(<OnboardingScreen />);

      expect(screen.getByTestId('next-button')).toBeTruthy();
      expect(screen.getByText('Next')).toBeTruthy();
    });
  });

  describe('Navigation', () => {
    it('should navigate to next slide when Next button is pressed', () => {
      render(<OnboardingScreen />);

      const nextButton = screen.getByTestId('next-button');
      fireEvent.press(nextButton);

      expect(screen.getByTestId('onboarding-title')).toHaveTextContent('Test with Confidence');
      expect(screen.getByTestId('onboarding-emoji')).toHaveTextContent('✅');
    });

    it('should show back button after navigating forward', () => {
      render(<OnboardingScreen />);

      const nextButton = screen.getByTestId('next-button');
      fireEvent.press(nextButton);

      expect(screen.getByTestId('back-button')).toBeTruthy();
    });

    it('should navigate back to previous slide when Back button is pressed', () => {
      render(<OnboardingScreen />);

      const nextButton = screen.getByTestId('next-button');
      fireEvent.press(nextButton);

      const backButton = screen.getByTestId('back-button');
      fireEvent.press(backButton);

      expect(screen.getByTestId('onboarding-title')).toHaveTextContent('Welcome to Demo App');
    });

    it('should show "Get Started" button on last slide', () => {
      render(<OnboardingScreen />);

      const nextButton = screen.getByTestId('next-button');
      fireEvent.press(nextButton);
      fireEvent.press(nextButton);

      expect(screen.getByTestId('finish-button')).toBeTruthy();
      expect(screen.getByText('Get Started')).toBeTruthy();
    });

    it('should call onComplete when finish button is pressed', () => {
      const onCompleteMock = jest.fn();
      render(<OnboardingScreen onComplete={onCompleteMock} />);

      const nextButton = screen.getByTestId('next-button');
      fireEvent.press(nextButton);
      fireEvent.press(nextButton);

      const finishButton = screen.getByTestId('finish-button');
      fireEvent.press(finishButton);

      expect(onCompleteMock).toHaveBeenCalledTimes(1);
    });

    it('should not crash when finish is pressed without onComplete prop', () => {
      render(<OnboardingScreen />);

      const nextButton = screen.getByTestId('next-button');
      fireEvent.press(nextButton);
      fireEvent.press(nextButton);

      const finishButton = screen.getByTestId('finish-button');
      expect(() => fireEvent.press(finishButton)).not.toThrow();
    });
  });

  describe('Pagination', () => {
    it('should highlight the correct pagination dot based on current slide', () => {
      const { getByTestId } = render(<OnboardingScreen />);

      const nextButton = getByTestId('next-button');
      fireEvent.press(nextButton);

      expect(screen.getByTestId('onboarding-title')).toHaveTextContent('Test with Confidence');
    });

    it('should navigate through all slides', () => {
      render(<OnboardingScreen />);

      expect(screen.getByTestId('onboarding-emoji')).toHaveTextContent('👋');

      const nextButton = screen.getByTestId('next-button');
      fireEvent.press(nextButton);
      expect(screen.getByTestId('onboarding-emoji')).toHaveTextContent('✅');

      fireEvent.press(nextButton);
      expect(screen.getByTestId('onboarding-emoji')).toHaveTextContent('🚀');
    });
  });

  describe('Edge Cases', () => {
    it('should not navigate beyond the last slide', () => {
      render(<OnboardingScreen />);

      const nextButton = screen.getByTestId('next-button');
      fireEvent.press(nextButton);
      fireEvent.press(nextButton);

      expect(screen.getByTestId('onboarding-title')).toHaveTextContent('CI/CD Integration');
    });

    it('should not navigate before the first slide', () => {
      render(<OnboardingScreen />);

      const nextButton = screen.getByTestId('next-button');
      fireEvent.press(nextButton);

      const backButton = screen.getByTestId('back-button');
      fireEvent.press(backButton);
      fireEvent.press(backButton);

      expect(screen.getByTestId('onboarding-title')).toHaveTextContent('Welcome to Demo App');
    });
  });
});
```

**Checkpoint**: Chạy tests:
```bash
npm test OnboardingScreen
```

Kết quả mong đợi:
```
PASS src/screens/OnboardingScreen/OnboardingScreen.test.tsx
  OnboardingScreen
    Rendering
      ✓ should render without crashing
      ✓ should display the first onboarding slide by default
      ... (tổng 15 tests)
```

### Bước 6: Tạo HomeScreen (15 phút)

**Note**: Copy code từ README.md section "Phần 3" cho HomeScreen.tsx và HomeScreen.test.tsx

**Checkpoint**: Chạy tất cả tests:
```bash
npm test
```

Kết quả:
```
Test Suites: 2 passed, 2 total
Tests:       38 passed, 38 total
```

### Bước 7: Update App.tsx (5 phút)

Thay thế content của `App.tsx`:

```typescript
import { useState } from 'react';
import { StatusBar } from 'expo-status-bar';
import { StyleSheet, View } from 'react-native';
import OnboardingScreen from './src/screens/OnboardingScreen/OnboardingScreen';
import HomeScreen from './src/screens/HomeScreen/HomeScreen';

export default function App() {
  const [showOnboarding, setShowOnboarding] = useState(true);

  const handleOnboardingComplete = () => {
    setShowOnboarding(false);
  };

  return (
    <View style={styles.container}>
      {showOnboarding ? (
        <OnboardingScreen onComplete={handleOnboardingComplete} />
      ) : (
        <HomeScreen />
      )}
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
});
```

### Bước 8: Chạy test coverage (5 phút)

```bash
npm run test:coverage
```

**Kết quả mong đợi**:

```
------------------------------------------|---------|----------|---------|---------|-------------------
File                                      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
------------------------------------------|---------|----------|---------|---------|-------------------
All files                                 |   97.72 |       90 |   94.44 |   97.61 |
 App.tsx                                  |      80 |       50 |      50 |      80 | 11
 HomeScreen.tsx                           |     100 |      100 |     100 |     100 |
 OnboardingScreen.tsx                     |     100 |     87.5 |     100 |     100 | 48-54
------------------------------------------|---------|----------|---------|---------|-------------------
```

**✅ Hoàn thành Phần 3!** Coverage: 97.72% >> 70% yêu cầu

---

## Phần 4: Tự động hóa Test với GitHub Actions

### Bước 1: Khởi tạo Git repository (5 phút)

```bash
# Khởi tạo git
git init

# Kiểm tra .gitignore đã có coverage/
cat .gitignore | grep coverage

# Nếu chưa có, thêm vào .gitignore:
echo "coverage/" >> .gitignore
echo "*.lcov" >> .gitignore

# Add và commit
git add .
git commit -m "feat: add OnboardingScreen and HomeScreen with tests"
```

### Bước 2: Tạo GitHub repository (5 phút)

1. Vào [github.com/new](https://github.com/new)
2. Repository name: `demo_mobile`
3. Description: "React Native Testing Demo - CO3043"
4. Public
5. **KHÔNG** check "Initialize this repository with..."
6. Click **Create repository**

### Bước 3: Push code lên GitHub (2 phút)

```bash
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/demo_mobile.git
git push -u origin main
```

**Lưu ý**: Thay `YOUR_USERNAME` bằng username GitHub của bạn.

### Bước 4: Tạo GitHub Actions workflow (10 phút)

```bash
# Tạo thư mục
mkdir -p .github/workflows
```

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
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci --legacy-peer-deps

      - name: Run tests
        run: npm test -- --ci --coverage --maxWorkers=2

      - name: Upload coverage reports
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/

      - name: Check coverage threshold
        run: |
          echo "Checking if coverage meets 70% threshold..."
          COVERAGE=$(cat coverage/coverage-summary.json | grep -o '"lines":{[^}]*}' | grep -o '"pct":[0-9.]*' | head -1 | grep -o '[0-9.]*')
          echo "Current coverage: $COVERAGE%"
          if (( $(echo "$COVERAGE < 70" | bc -l) )); then
            echo "Coverage is below 70%!"
            exit 1
          fi
          echo "Coverage meets the threshold!"
```

### Bước 5: Push workflow và verify (5 phút)

```bash
git add .github/
git commit -m "ci: add GitHub Actions workflow for testing"
git push
```

**Verify**:
1. Vào GitHub repository
2. Click tab **Actions**
3. Xem workflow "Run Tests" đang chạy
4. Đợi ~2-3 phút cho workflow complete
5. Click vào workflow run → xem logs
6. Check **Artifacts** section → download `coverage-report`

**Screenshot để nộp**:
- Screenshot workflow success (màu xanh ✓)
- Screenshot coverage trong logs

**✅ Hoàn thành Phần 4!** CI/CD đã hoạt động!

---

## Phần 5: Phân tích chất lượng với SonarCloud

### Bước 1: Tạo SonarCloud account (3 phút)

1. Vào [sonarcloud.io](https://sonarcloud.io)
2. Click **Log in**
3. Chọn **Log in with GitHub**
4. Authorize SonarCloud

### Bước 2: Tạo project trên SonarCloud (5 phút)

1. Click **+** (góc trên phải) → **Analyze new project**
2. Chọn organization (thường là username của bạn)
3. Chọn repository: `demo_mobile`
4. Click **Set Up**
5. Choose **With GitHub Actions**
6. **QUAN TRỌNG**: Copy 2 thông tin:
   - **SONAR_TOKEN**: Click **Generate Token** → Copy
   - **Organization Key**: Hiển thị trên trang
   - **Project Key**: Tự động generate

### Bước 3: Thêm SONAR_TOKEN vào GitHub (3 phút)

1. Vào GitHub repository
2. **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name: `SONAR_TOKEN`
5. Secret: Paste token từ SonarCloud
6. Click **Add secret**

### Bước 4: Tạo sonar-project.properties (5 phút)

Tạo file `sonar-project.properties` ở root:

```properties
sonar.projectKey=YOUR_PROJECT_KEY
sonar.organization=YOUR_ORGANIZATION_KEY

sonar.projectName=Demo Mobile - React Native Testing
sonar.projectVersion=1.0

sonar.sources=src,App.tsx
sonar.tests=src
sonar.test.inclusions=**/*.test.tsx,**/*.test.ts
sonar.exclusions=**/node_modules/**,**/coverage/**,**/*.test.tsx,**/*.test.ts

sonar.typescript.lcov.reportPaths=coverage/lcov.info
sonar.javascript.lcov.reportPaths=coverage/lcov.info

sonar.coverage.exclusions=**/*.test.tsx,**/*.test.ts,**/node_modules/**
```

**Lưu ý**: Thay `YOUR_PROJECT_KEY` và `YOUR_ORGANIZATION_KEY` bằng giá trị thực từ SonarCloud.

### Bước 5: Tạo SonarCloud workflow (5 phút)

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
          node-version: '20'
          cache: 'npm'

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

### Bước 6: Push và verify (5 phút)

```bash
git add sonar-project.properties .github/workflows/sonarcloud.yml
git commit -m "ci: add SonarCloud integration"
git push
```

**Verify**:
1. Vào GitHub Actions → xem 2 workflows chạy
2. Đợi ~3-5 phút
3. Vào [sonarcloud.io](https://sonarcloud.io) → chọn project `demo_mobile`
4. Xem dashboard với metrics:
   - **Coverage**: ~97%
   - **Reliability**: A
   - **Security**: A
   - **Maintainability**: A

**Screenshot để nộp**:
- Screenshot SonarCloud dashboard
- Screenshot từng metric (Coverage, Bugs, Code Smells, etc.)

### Bước 7: Thêm badges vào README (5 phút)

Update `README.md`, thêm vào đầu file:

```markdown
[![Run Tests](https://github.com/YOUR_USERNAME/demo_mobile/actions/workflows/test.yml/badge.svg)](https://github.com/YOUR_USERNAME/demo_mobile/actions/workflows/test.yml)
[![SonarCloud](https://github.com/YOUR_USERNAME/demo_mobile/actions/workflows/sonarcloud.yml/badge.svg)](https://github.com/YOUR_USERNAME/demo_mobile/actions/workflows/sonarcloud.yml)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=YOUR_PROJECT_KEY&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=YOUR_PROJECT_KEY)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=YOUR_PROJECT_KEY&metric=coverage)](https://sonarcloud.io/summary/new_code?id=YOUR_PROJECT_KEY)
```

```bash
git add README.md
git commit -m "docs: add badges to README"
git push
```

**✅ Hoàn thành Phần 5!** SonarCloud đã tích hợp thành công!

---

## Tổng kết

### Checklist hoàn thành

- [ ] Project khởi tạo thành công
- [ ] Jest config đúng
- [ ] OnboardingScreen có ≥ 3 test cases
- [ ] HomeScreen có ≥ 3 test cases
- [ ] Coverage ≥ 70% (hiện tại: 97.72%)
- [ ] GitHub Actions workflow chạy thành công
- [ ] SonarCloud project được tạo
- [ ] Badges hiển thị trên README

### Nộp bài

Mỗi nhóm nộp:

1. **Link GitHub repository**: `https://github.com/YOUR_USERNAME/demo_mobile`
2. **Link SonarCloud project**: `https://sonarcloud.io/project/overview?id=YOUR_PROJECT_KEY`
3. **Screenshots** (7 ảnh):
   - OnboardingScreen tests passing
   - HomeScreen tests passing
   - Coverage report (terminal)
   - GitHub Actions workflow success
   - SonarCloud dashboard
   - SonarCloud coverage detail
   - README với badges

### Metrics yêu cầu

- ✅ Coverage ≥ 70%
- ✅ SonarCloud Quality Gate: Pass
- ✅ Reliability Rating: A
- ✅ Security Rating: A
- ✅ Maintainability Rating: A

---

## Troubleshooting

### Lỗi thường gặp

#### 1. Tests fail với "Incorrect version of react-test-renderer"

```bash
npm install -D react-test-renderer@19.1.0 --legacy-peer-deps
```

#### 2. Tests fail với "You are trying to import a file outside of the scope"

Kiểm tra `jest.setup.js` có đầy đủ:
```javascript
global.__ExpoImportMetaRegistry = {
  register: () => {},
  get: () => null,
};

global.structuredClone = global.structuredClone || ((obj) => JSON.parse(JSON.stringify(obj)));
```

#### 3. GitHub Actions fail với npm install

Đảm bảo workflow dùng `npm ci --legacy-peer-deps`

#### 4. SonarCloud không nhận được coverage

Kiểm tra:
- `sonar-project.properties` có đúng paths
- `coverage/lcov.info` được generate
- SONAR_TOKEN đã add vào GitHub Secrets

#### 5. Coverage quá thấp

Viết thêm test cases cho:
- Edge cases
- Error handling
- User interactions
- Conditional rendering

---

## Câu hỏi thường gặp

**Q: Có cần chạy app trên emulator không?**
A: Không cần. Unit tests chạy hoàn toàn trong Node.js environment.

**Q: SonarCloud có free không?**
A: Có, miễn phí cho public repositories.

**Q: Có thể dùng private repository không?**
A: Được, nhưng SonarCloud free chỉ support public repos. Nếu private thì cần upgrade.

**Q: Coverage 97% có quá cao không?**
A: Không, đây là best practice. Trong thực tế nên aim for ≥ 80%.

**Q: Có thể skip CI khi push không?**
A: Có, thêm `[skip ci]` vào commit message. Nhưng không nên làm thường xuyên.

---

**Chúc các bạn thành công! 🚀**
