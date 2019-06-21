# Test ứng dụng javascript với Jest Framework
## Jest là gì?
Jest là một JS framework khá thú vị và đơn giản được dùng để viết unit testing. Nó có thể làm việc tốt trên: React, Angular, Vue, Javascipt, Typescript
## Jest có gì hay ho?
Jest là một thư viện testing được tạo bởi facebook và được các công ty lớn như Twitter, Instagram, Spotify,... sử dụng. Nó có các ưu điểm sau:
- Nhanh, tối ưu
- Cấu hình đơn giản
- Tuỳ chọn Code coverage 
- Mocking dễ dàng
- Hỗ trợ snapshots
- Document chi tiết, dễ hiểu
## Cài Jest về máy
Tôi tạo một thư mục mới và chạy `npm init` (cài node tại đây: https://nodejs.org/en/ nếu trên máy bạn chưa cài) 
- Dùng `npm`: `npm install --save-dev jest`
- hoặc `yarn`: `yarn add --dev jest` để cài jest về máy. Vậy là bạn đã xong phần chuẩn bị rồi đấy
## Viết test case đơn giản
Tôi sẽ viết test case đơn giản cho việc công 2 số với nhau, tôi tạo mới file `app.js` chưa đoạn code thực hiện việc tính toán như sau:
```javascript
function sum(a, b) {
  return a + b;
}
module.exports = sum;
```
Sau đó, tạo file `app.test.js`, chưa các đoạn code test case
```javascript
const sum = require('./app');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```
Thêm đọan sau để set lệnh chạy test
```js
{
  "scripts": {
    "test": "jest"
  }
}
```
Cuối cùng, trên terminal, bạn chạy lệnh `yarn test` hoặc `npm run test`. Trên console sẽ in ra màn hình như sau nếu thành công:
```
PASS  ./app.test.js
✓ adds 1 + 2 to equal 3 (5ms)
```
## Các tính năng khác
Trên đây là 1 ví dụ cơ bản và test case đơn giản, để thực hiện test các test case phức tạp hơn thì dưới đây là các tính năng của Jest:

- Using Matchers
```javascript
test('two plus two is four', () => {
  expect(2 + 2).toBe(4);
});
test('object assignment', () => {
  const data = {one: 1};
  data['two'] = 2;
  expect(data).toEqual({one: 1, two: 2});
});
```
- Test Truthiness (kiểm tra tính đúng đắn)
```javascript
test('null', () => {
  const n = null;
  expect(n).toBeNull();
  expect(n).toBeDefined();
  expect(n).not.toBeUndefined();
  expect(n).not.toBeTruthy();
  expect(n).toBeFalsy();
});

test('zero', () => {
  const z = 0;
  expect(z).not.toBeNull();
  expect(z).toBeDefined();
  expect(z).not.toBeUndefined();
  expect(z).not.toBeTruthy();
  expect(z).toBeFalsy();
});
```
- Number:
```javascript
test('two plus two', () => {
  const value = 2 + 2;
  expect(value).toBeGreaterThan(3);
  expect(value).toBeGreaterThanOrEqual(3.5);
  expect(value).toBeLessThan(5);
  expect(value).toBeLessThanOrEqual(4.5);

  // toBe and toEqual are equivalent for numbers
  expect(value).toBe(4);
  expect(value).toEqual(4);
});
```
- String:
```javascript
test('there is no I in team', () => {
  expect('team').not.toMatch(/I/);
});

test('but there is a "stop" in Christoph', () => {
  expect('Christoph').toMatch(/stop/);
});
```
- Array:
```javascript
const shoppingList = [
  'diapers',
  'kleenex', 
  'trash bags', 
  'paper towels', 
  'beer',
];

test('the shopping list has beer on it', () => {
  expect(shoppingList).toContain('beer');
});
```
- Exception:
```javascript
function compileAndroidCode() {
  throw new ConfigError('you are using the wrong JDK');
}

test('compiling android goes as expected', () => {
  expect(compileAndroidCode).toThrow();
  expect(compileAndroidCode).toThrow(ConfigError);

  // You can also use the exact error message or a regexp
  expect(compileAndroidCode).toThrow('you are using the wrong JDK');
  expect(compileAndroidCode).toThrow(/JDK/);
});
```
- Callbacks
```javascript
test('the data is peanut butter', done => {
  function callback(data) {
    expect(data).toBe('peanut butter');
    done();
  }

  fetchData(callback);
});
- Promise:
test('the data is peanut butter', () => {
  expect.assertions(1);
  return fetchData().then(data => {
    expect(data).toBe('peanut butter');
  });
});
```
- Async/Await
```javascript
test('the data is peanut butter', async () => {
  expect.assertions(1);
  const data = await fetchData();
  expect(data).toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  expect.assertions(1);
  try {
    await fetchData();
  } catch (e) {
    expect(e).toMatch('error');
  }
});
```
- Setup cho nhiều test:
```javascript
beforeEach(() => {
  initializeCityDatabase();
});

afterEach(() => {
  clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```
- Setup một lần (bắt đầu hoặc kết thúc):
```javascript
beforeAll(() => {
  return initializeCityDatabase();
});

afterAll(() => {
  return clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```
- Nhóm test lại (describe):
```javascript
// Applies to all tests in this file
beforeEach(() => {
  return initializeCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});

describe('matching cities to foods', () => {
  // Applies only to tests in this describe block
  beforeEach(() => {
    return initializeFoodDatabase();
  });

  test('Vienna <3 sausage', () => {
    expect(isValidCityFoodPair('Vienna', 'Wiener Schnitzel')).toBe(true);
  });

  test('San Juan <3 plantains', () => {
    expect(isValidCityFoodPair('San Juan', 'Mofongo')).toBe(true);
  });
});
```
- Dùng mock function:
```javascript
function forEach(items, callback) {
  for (let index = 0; index < items.length; index++) {
    callback(items[index]);
  }
}
const mockCallback = jest.fn();
forEach([0, 1], mockCallback);

// The mock function is called twice
expect(mockCallback.mock.calls.length).toBe(2);

// The first argument of the first call to the function was 0
expect(mockCallback.mock.calls[0][0]).toBe(0);

// The first argument of the second call to the function was 1
expect(mockCallback.mock.calls[1][0]).toBe(1);
```
- Mock return value (define giá trị trả lại của function)
```javascript
const myMock = jest.fn();
console.log(myMock());
// > undefined

myMock.mockReturnValueOnce(10)
 .mockReturnValueOnce('x')
 .mockReturnValue(true);

console.log(myMock(), myMock(), myMock(), myMock());
// > 10, 'x', true, true
```
- Custom Matchers
```javascript
// The mock function was called at least once
expect(mockFunc).toBeCalled();

// The mock function was called at least once with the specified args
expect(mockFunc).toBeCalledWith(arg1, arg2);

// The last call to the mock function was called with the specified args
expect(mockFunc).lastCalledWith(arg1, arg2);
```
- Dùng thuộc tính .mock để inspect nhiều xác định hơn:
```javascript
// The mock function was called at least once
expect(mockFunc.mock.calls.length).toBeGreaterThan(0);

// The mock function was called at least once with the specified args
expect(mockFunc.mock.calls).toContain([arg1, arg2]);

// The last call to the mock function was called with the specified args
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1]).toEqual(
  [arg1, arg2]
);

// The first arg of the last call to the mock function was `42`
// (note that there is no sugar helper for this specific of an assertion)
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1][0]).toBe(42);
```
- Mock Implementations: Hơn cả chức năng trả lại, nó còn thay thế toàn bộ hàm mocking:
```javascript
const myMockFn = jest.fn(() => 'default')
  .mockImplementationOnce(() => 'first call')
  .mockImplementationOnce(() => 'second call');

console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());

// > 'first call', 'second call', 'default', 'default'
```
- Testing snapshot:
```javascript
it('renders a snapshot', () => {
  const tree = renderer.create(<App/>).toJSON();
  expect(tree).toMatchSnapshot();
});
```
## Kết luận
Trên đây là các tính năng và ví dụ đơn giản về Jest mà tôi muốn giới thiệu với các bạn. Hy vọng các bạn đã đọc được bài viết bổ ích.
Xin cảm ơn.



































