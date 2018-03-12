배열에는 `정적`배열과 `동적`배열 두 종류가 있다.

```js
unit[2] fixedArray; //2개의 원소를 담을 수 있는 고정길이 배열

string[5] stringArray; //또다른 고정 배열으로 5개의 스트링을 담을 수 있다. 

unit[] dynamicArray; //동적배열은 고정된 크기가 없으며 계속 크기가 커질 수 있다.
```

구조체 배열을 생성할 수 있다.

```js
Person[] people: //이는 동적 배열로 원소를 계속 추가 가능함
```

`Public`으로 배열을 선언 가능하다. 이 경우는 자동으로 `getter`함수가 추가된다.

```js
Person[] public people //public이 중간에 붙는 걸 유의하자
```

배열에 추가를 하는 경우

```
Person satoshi = Person(172, "Satoshi");
people.push(satoshi) //이런식으로 추가 가능하다.
```

함수 선언은 다음과 같다.

```
function eat(string _name, uint _amount) {
    //TODO
}

eat('hambuger',1000); //함수의 실행
```

`private`으로도 생성을 할 수 있다.

```
function addToArray(uint _number) private {
  numbers.push(_number)
}
```

반환값 반환하는 경우

```
string greeting = "what's up dog"

function sayHello() public returns (string) { //string 타입으로 리턴합니다. 
  return greeting;
}
```

함수제어자의 경우 `view`와 `pure`를 가지고 있다.

view의 경우 함수가 데이터를 보기만 하고 변경하지 않는 다는 뜻이다.

pure는 어떤 데이터도 접근하지 않는 것을 의미한다. 앱에서 읽지도 않고, 반환값이 함수에 전달된 인자값에 따라서 달라진다.

> 솔리디티 컴파일러는 이러한 부분에 대해서 무엇을 쓰면 좋을지 경고를 표시해준다.

```js
function sayHello() public view returns (string) {

}

function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
```

**랜덤문자 생성**

솔리디티에서 랜덤 발생하는 방법은 SHA3의 한 종류인 `keccak256`을 이용하면 된다.

> 완벽하고 안전한 난수 발생기는 아닌걸 유의하자.

```
//6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5
keccak256("aaaab");
//b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9
keccak256("aaaac");
```

위 글자 하나때문에 값이 완전히 달라지는 걸 유념하자.

**형 변환**

가끔씩 형병환 할 경우가 있다.

```
uint a = 5;
uint b = 6;
// a * b가 uint8이 아닌 uint를 반환하기 때문에 에러메세지가 나옴
uint8 c = a * b
// b를 unit8 으로 형변환해서 코드가 제대로 동작하기 위해서...
uint8 c = a * uint8(b);
```

**이벤트**

이벤트는 계약이 블록체인 상에서 어떠한 이벤트 발생시 의사소통 하는 방법을 말한다.

계약은 특정이벤트가 발생되는지 리스닝하는 걸 뜻한다.

```
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public {
    uint result = _x + _y;
    //이벤트를 실행하여 앱에게 add 함수가 실행되었음을 알린다. 
    IntegersAdded(_x, _y, result);
    return result;
}

//////////////// 호출은
MyContract.IntegerAdded(function(error,result){
    //결과와 관련된 행동을 취한다.    
});
```

최종 소스

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string _name, uint _dna) private {        
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        NewZombie(id, _name, _dna);
    } 

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```



**Web3 연결 코드**

```js
// 여기에 우리가 만든 컨트랙트에 접근하는 방법을 제시한다:
var abi = /* abi generated by the compiler */
var ZombieFactoryContract = web3.eth.contract(abi)
var contractAddress = /* our contract address on Ethereum after deploying */
var ZombieFactory = ZombieFactoryContract.at(contractAddress)
// `ZombieFactory`는 우리 컨트랙트의 public 함수와 이벤트에 접근할 수 있다.

// 일종의 이벤트 리스너가 텍스트 입력값을 취한다:
$("#ourButton").click(function(e) {
  var name = $("#nameInput").val()
  // 우리 컨트랙트의 `createRandomZombie`함수를 호출한다:
  ZombieFactory.createRandomZombie(name)
})

// `NewZombie` 이벤트가 발생하면 사용자 인터페이스를 업데이트한다
var event = ZombieFactory.NewZombie(function(error, result) {
  if (error) return
  generateZombie(result.zombieId, result.name, result.dna)
})

// 좀비 DNA 값을 받아서 이미지를 업데이트한다
function generateZombie(id, name, dna) {
  let dnaStr = String(dna)
  // DNA 값이 16자리 수보다 작은 경우 앞 자리를 0으로 채운다
  while (dnaStr.length < 16)
    dnaStr = "0" + dnaStr

  let zombieDetails = {
    // 첫 2자리는 머리의 타입을 결정한다. 머리 타입에는 7가지가 있다. 그래서 모듈로(%) 7 연산을 하여
    // 0에서 6 중 하나의 값을 얻고 여기에 1을 더해서 1에서 7까지의 숫자를 만든다. 
    // 이를 기초로 "head1.png"에서 "head7.png" 중 하나의 이미지를 불러온다:
    headChoice: dnaStr.substring(0, 2) % 7 + 1,
    // 두번째 2자리는 눈 모양을 결정한다. 눈 모양에는 11가지가 있다:
    eyeChoice: dnaStr.substring(2, 4) % 11 + 1,
    // 셔츠 타입에는 6가지가 있다:
    shirtChoice: dnaStr.substring(4, 6) % 6 + 1,
    // 마지막 6자리는 색깔을 결정하며, 360도(degree)까지 지원하는 CSS의 "filter: hue-rotate"를 이용하여 아래와 같이 업데이트된다:
    skinColorChoice: parseInt(dnaStr.substring(6, 8) / 100 * 360),
    eyeColorChoice: parseInt(dnaStr.substring(8, 10) / 100 * 360),
    clothesColorChoice: parseInt(dnaStr.substring(10, 12) / 100 * 360),
    zombieName: name,
    zombieDescription: "A Level 1 CryptoZombie",
  }
  return zombieDetails
}
```

최종 나의 좀비

```
https://share.cryptozombies.io/ko/lesson/1/share/tommy
```


