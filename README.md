Snaaaaaake
===
DEMO
---
https://chendaryen.github.io/Snaaaaaake/

資料定義
---
```JS
data: {
    score: 當前分數,
    best: 歷史高分,
    snakeSpeed: 貪吃蛇速度(更新時間),
    direction: [貪吃蛇移動方向],
    situation: '遊戲狀態',
    alive: 貪吃蛇是否存活,
    positionRecord: [紀錄貪吃蛇移動軌跡],
    pointPosition: {
        x: 星星x座標,
        y: 星星y座標
    },
}
```
移動
---
```JS
move(direction) {
    let vm = this;
    let snakeLength = vm.positionRecord.length;
    let headPos = vm.positionRecord[snakeLength - 1];
    let tailPos = vm.positionRecord[0];
    document.querySelectorAll('.row')[tailPos.y].children[tailPos.x].children[0].style.backgroundColor = '';
    let newHeadPos = {
        x: (headPos.x + direction[0] + 28) % 28,            //穿牆術
        y: (headPos.y + direction[1] + 16) % 16,
    };
    vm.positionRecord.push(newHeadPos);
    if(newHeadPos.x != vm.pointPosition.x || newHeadPos.y != vm.pointPosition.y) {            //確認是否得分
        vm.positionRecord.splice(0, 1);
    }else {
        vm.score ++;
        vm.cancelPoint();
        vm.randomPoint();
    };
    vm.drawSnake();
}
```
繪製貪吃蛇
---
```JS
drawSnake() {
    let vm = this;
    if(vm.alive) {
        let rows = document.querySelectorAll('.row');
        let snakeLength = vm.positionRecord.length;
        for(i=0; i<snakeLength - 1; i++) {
            let opacity = (i+2) / snakeLength;
            rows[vm.positionRecord[i].y].children[vm.positionRecord[i].x].children[0].style.backgroundColor = `rgba(255, 255, 255, ${opacity})`;
        };
        rows[vm.positionRecord[snakeLength - 1].y].children[vm.positionRecord[snakeLength - 1].x].children[0].style.backgroundColor = '#00FFE2';
        for(i=0; i<snakeLength-4; i++) {
            if(vm.positionRecord[snakeLength - 1].x == vm.positionRecord[i].x && vm.positionRecord[snakeLength - 1].y == vm.positionRecord[i].y) {            //貪吃蛇咬中身體即死亡
                vm.alive = false;
                rows[vm.positionRecord[snakeLength - 1].y].children[vm.positionRecord[snakeLength - 1].x].children[0].style.backgroundColor = '#ff7600';
                setTimeout(function() {
                    vm.situation = 'end';
                    if(vm.score > vm.best) {
                        vm.best = vm.score;
                        localStorage.setItem('bestScore', vm.best);
                    };
                    vm.$nextTick(function() {
                        document.querySelector('body').onkeydown = vm.replay
                    })
                }, 1500);
            }
        };    
    }else {
        clearInterval(vm.interval);
    }

},
```
隨機製造星星
---
```JS
randomPoint() {
    let rows = document.querySelectorAll('.row');
    let randomX = Math.floor(Math.random() * 28);
    let randomY = Math.floor(Math.random() * 16);
    let newPoint = true;
    let snakeLength =  this.positionRecord.length;
    for(i=0; i<snakeLength; i++) {            //判斷新產生的位置是否與蛇身重疊
        if(randomX == this.positionRecord[i].x && randomY == this.positionRecord[i].y) {
            newPoint = false;
            break;
        }
    };
    if(newPoint) {
        rows[randomY].children[randomX].style.backgroundImage = 'url("image/ic-point.svg")';
        rows[randomY].children[randomX].style.backgroundColor = 'rgba(255, 255, 255, 0.2)';
        for(i=1; i<8; i++) {
            let opacity = (1 - (i)/8) * 0.2;
            if(randomX - i >= 0 && randomX - i <= 27) {
                rows[randomY].children[randomX - i].style.backgroundColor = `rgba(255, 255, 255, ${opacity})`;
            };
            if(randomX + i >= 0 && randomX + i <= 27) {
                rows[randomY].children[randomX + i].style.backgroundColor = `rgba(255, 255, 255, ${opacity})`;
            };
            if(randomY - i >= 0 && randomY - i <= 15) {
                rows[randomY - i].children[randomX].style.backgroundColor = `rgba(255, 255, 255, ${opacity})`;
            };
            if(randomY + i >= 0 && randomY + i <= 15) {
                rows[randomY + i].children[randomX].style.backgroundColor = `rgba(255, 255, 255, ${opacity})`;
            };
        };
        this.pointPosition = {
            x: randomX,
            y: randomY
        };
    }else {           //若新產生的位置與蛇身重疊，再次呼叫自己產生新位置
        this.randomPoint();
    };
},
```
