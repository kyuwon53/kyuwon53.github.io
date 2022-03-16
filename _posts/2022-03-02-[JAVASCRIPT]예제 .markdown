function printName() {
  var inner1 = () => console.log(this.name, name);
  var inner2 = function () {console.log(this.name, name)};
  function print() {
    inner1();
    inner2();
  }

  console.log(name);
  var name = 'C';

  print();
  name = 'D';
  return print;
}
name = 'B';
({name: 'A', fnc: printName}).fnc()();

// 출력 결과를 아래 기술하세요