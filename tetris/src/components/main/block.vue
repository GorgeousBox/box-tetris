<template>
  <div id="Block">
    <div class="block-home"
         v-show="!isGame">
      <p>俄罗斯方块</p>
      <li @click="isGameHome(true, 700)">休闲难度</li>
      <li @click="isGameHome(true, 200)">普通难度</li>
      <li @click="isGameHome(true, 100)">单身难度</li>
    </div>

    <div class="block-game"
         v-show="isGame">
      <div class="block-game-head">
        <div class="block-game-head-menu"
             @click="isGameStop()">菜单</div>
        <div>加速：S，改变：W，移动：A、D，菜单：ESC</div>
      </div>
      <div class="block-game-gameRegion">
        <div v-for="(item,value) in blockAll"
             :key="value">
          <li v-for="(item2,value2) in blockAll[value]"
              :key="value2"></li>
        </div>
      </div>
      <div class="block-game-nextRegion">
        <div class="block-game-nextRegion-next">
          <p>下一个小方块</p>
          <div class="block-game-nextRegion-next-all">
            <div v-for="(item,value) in blockNextAll"
                 :key="value">
              <li v-for="(item,value) in blockNextAll[0]"
                  :key="value"></li>
            </div>
          </div>
        </div>
        <div class="block-game-nextRegion-scorePanel">
          <p>已消除行数：{{numberRows}}</p>
        </div>
      </div>
    </div>

    <div class="block-stop"
         v-show="isStop">
      <div class="block-stop-page">
        <div @click="isGameStop()">返回游戏</div>
        <div @click="isGameHome(false)">主菜单</div>
      </div>
    </div>

    <div class="block-gameOver"
         v-show="gameOver">
      <p>GAME OVER</p>
      <div>
        <div @click="isGameHome(false)">主菜单</div>
        <p>本次消除行数 : {{numberRows}}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      isGame: false,// 是否显示游戏页面，false为主页，true为游戏页面。 
      isStop: false,// 是否显示暂停页面
      difficulty: 0,// 记录当前难度数据
      difficultyKey: 0,// 键盘s改变时间
      numberRows: 0,// 记录消除行数据
      currentBlock: [],// 当前下落的方块
      currentBlockG: 0,// 当前下落的方块的形状
      nextBlock: [],// 下一个方块
      blockTimer: null,// 方块下落定时器
      blockX: 5,// 方块当前x位置
      blockY: 0,// 方块当前y位置
      blockXB: 0,// 方块x轴总长度
      blockYB: 0,// 方块y轴总长度
      gameOver: false,// 游戏是否结束
      blockAll: [ // 总方块数据
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      ],

      block: { // 方块形状数据
        // 方形: 2*2
        // 1种情况
        q: [
          [
            [0, 0],
            [0, 1],
            [1, 0],
            [1, 1]
          ],
        ],
        // 长条: 1*4
        // 2种
        l: [
          [
            [0, 0],
            [0, 1],
            [0, 2],
            [0, 3],
          ],
          // 横向长条
          [
            [0, 0],
            [1, 0],
            [2, 0],
            [3, 0],
          ]
        ],
        // S形:  2*3
        // 2种
        s1: [
          [
            [0, 0],
            [1, 0],
            [1, 1],
            [2, 1],
          ],
          // 竖向s1
          [
            [1, 0],
            [0, 1],
            [1, 1],
            [0, 2],
          ]
        ],
        // S形:  2*3 镜像
        // 2种
        s2: [
          [
            [0, 1],
            [1, 0],
            [1, 1],
            [2, 0],
          ],
          // 竖向s2
          [
            [0, 0],
            [0, 1],
            [1, 1],
            [1, 2],
          ]
        ],
        // T形:  3*2
        // 4种
        t: [
          [
            [1, 0],
            [0, 1],
            [1, 1],
            [2, 1],
          ],
          // 右T
          [
            [0, 0],
            [0, 1],
            [1, 1],
            [0, 2],
          ],
          // 下T
          [
            [0, 0],
            [1, 0],
            [2, 0],
            [1, 1],
          ],
          // 左T
          [
            [1, 0],
            [0, 1],
            [1, 1],
            [1, 2],
          ],
        ],
        // L形:  3*2 左边高
        // 4种
        l1: [
          [
            [0, 1],
            [1, 1],
            [2, 1],
            [0, 0],
          ],
          // 右L1
          [
            [0, 0],
            [1, 0],
            [0, 1],
            [0, 2],
          ],
          // 下L1
          [
            [0, 0],
            [1, 0],
            [2, 0],
            [2, 1],
          ],
          // 左L1
          [
            [1, 0],
            [1, 1],
            [0, 2],
            [1, 2],
          ],
        ],
        // L形:  3*2 左边高
        // 4种
        l2: [
          [
            [0, 1],
            [1, 1],
            [2, 1],
            [2, 0],
          ],
          // 右L2
          [
            [0, 0],
            [0, 1],
            [0, 2],
            [1, 2],
          ],
          // 下L2
          [
            [0, 0],
            [1, 0],
            [2, 0],
            [0, 1],
          ],
          // 左L2
          [
            [0, 0],
            [1, 0],
            [1, 1],
            [1, 2],
          ],
        ],
      },

      blockNextAll: [ // 下一个方块总数据
        [0, 0, 0, 0],
        [0, 0, 0, 0],
        [0, 0, 0, 0],
        [0, 0, 0, 0],
      ],

      colorRandom: [],
      nextcolorRandom: [],
    };
  },

  methods: {
    // 暂停页面显示函数
    isGameStop () {
      this.isStop = !this.isStop;
      if (this.isStop) clearInterval(this.blockTimer)
      else this.blockDown();
    },
    // 主页面和游戏页面切换函数，并当处于游戏页面时进行run函数运行
    isGameHome (is, difficulty) {
      this.isGame = is;
      this.difficulty = difficulty;
      if (this.isGame) this.gameRun();
      else 
      {
        this.gameOver = false;
        this.isStop = false;
        this.gameReset();
      }
    },

    // 渲染方块函数
    blockRender () {
      this.currentBlock[this.currentBlockG].forEach(item => {
        let x = item[0] + this.blockX;
        let y = item[1] + this.blockY;
        const a = document.querySelector(`.block-game-gameRegion div:nth-child(${y + 1}) li:nth-child(${x + 1})`);
        // if (item2) a.classList.add("colorWhite");
        // else a.classList.remove("colorWhite");
        a.style.backgroundColor = `rgb(${this.colorRandom[0]},${this.colorRandom[1]},${this.colorRandom[2]})`;
      });

      this.blockAll.forEach((item, value) => {
        item.forEach((item2, value2) => {
          const temp = document.querySelector(`.block-game-gameRegion div:nth-child(${value + 1}) li:nth-child(${value2 + 1})`);
          // console.log(item2);
          if (!item2) temp.style.backgroundColor = "black";
        })
      });
    },

    // 下一个方块预测渲染函数
    nextBlockRender () {
      this.blockNextAll.forEach((item, value) => {
        item.forEach((item2, value2) => {
          const temp = document.querySelector(`.block-game-nextRegion-next-all div:nth-child(${value + 1}) li:nth-child(${value2 + 1})`);
          // if (item2) temp.classList.add("colorWhite");
          // else temp.classList.remove("colorWhite");
          if (item2) temp.style.backgroundColor = `rgb(${this.colorRandom[0]},${this.colorRandom[1]},${this.colorRandom[2]})`;
          else temp.style.backgroundColor = "rgb(0,0,0)";
        })
      })
    },

    // 随机方块函数
    blockRandom (is) {
      switch (is)
      {
        case 1: this.currentBlock = this.nextBlock; break;
        case 2:
          {
            const blockName = ["q", "l", "s1", "s2", "t", "l1", "l2"];
            const blockRandomNumber = Math.floor(Math.random() * (blockName.length));
            this.currentBlock = this.block[blockName[blockRandomNumber]];
          }; break;
        case 3:
          {
            const blockName = ["q", "l", "s1", "s2", "t", "l1", "l2"];
            const blockRandomNumber = Math.floor(Math.random() * (blockName.length));
            this.nextBlock = this.block[blockName[blockRandomNumber]];
          }
      }


      // this.blockRender();
      // console.log(this.blockYB,this.blockXB);
      // if (blockRandomNumber === 0) 
      // {
      //   this.blockXB = 1;
      //   this.blockYB = 1;
      // }
      // else if (blockRandomNumber === 1)
      // {
      //   this.blockXB = 0;
      //   this.blockYB = 3;
      // }
      // else if (blockRandomNumber === 2 || blockRandomNumber === 3 || blockRandomNumber === 4 || blockRandomNumber === 5 || blockRandomNumber === 6)
      // {
      //   this.blockXB = 2;
      //   this.blockYB = 1;
      // }
    },

    // 移动渲染方块函数
    blockMove (is = true) {
      if (is)
      {
        this.currentBlock[this.currentBlockG].forEach(item => {
          let x = item[0] + this.blockX;
          let y = item[1] + this.blockY;
          this.blockAll[y][x] = 1;
        });
        this.blockRender();
      }
      else
      {
        this.blockNextAll = [
          [0, 0, 0, 0],
          [0, 0, 0, 0],
          [0, 0, 0, 0],
          [0, 0, 0, 0],
        ];
        // console.log(this.nextBlock[0]);
        this.nextBlock[0].forEach(item => {
          let x = item[0];
          let y = item[1];
          this.blockNextAll[y][x] = 1;
          // console.log(this.blockNextAll);
        });
        this.nextBlockRender();
      }
    },

    // 清除上一次移动方块的函数
    blockClear () {
      this.currentBlock[this.currentBlockG].forEach(item => {
        let x = item[0] + this.blockX;
        let y = item[1] + this.blockY;
        this.blockAll[y][x] = 0;
      });
      this.blockRender();
    },

    // 用键盘驱动方块移动函数
    blockKeyboard (e) {
      // console.log(this.blockX, temp);
      this.blockClear();
      if (this.blockX <= 0) this.blockX = 0;
      if (e.key === "d" && this.blockIs(1, 0)) this.blockX++;
      if (e.key === "a" && this.blockIs(-1, 0)) this.blockX--;
      if (e.key === "s")
      {
        this.difficultyKey = 20;
        this.blockDown();
      }
      // console.log(String(this.blockX,this.blockY));
      if (
        e.key === "w" &&
        this.blockY + this.blockXB < this.blockAll.length &&
        this.blockIs(0, 1) &&
        this.blockIs(1, 1) &&
        this.blockIs(1, 0) &&
        this.blockIs(-1, 0)
      )
      {
        this.currentBlockG++;
        if (this.currentBlockG > this.currentBlock.length - 1) this.currentBlockG = 0;
      }
      this.gameBorder();
      this.blockMove();
    },

    // 判断游戏结界函数
    gameBorder () {
      let temp3 = [[], []];
      this.currentBlock[this.currentBlockG].forEach(item => {
        temp3[0].push(item[0]);
        temp3[1].push(item[1]);
      });
      this.blockXB = Math.max.apply(null, temp3[0]);
      this.blockYB = Math.max.apply(null, temp3[1]);

      const temp = this.blockAll[0].length - 1 - this.blockXB;
      const temp2 = this.blockAll.length - 1 - this.blockYB;
      if (this.blockX <= 0) this.blockX = 0;
      if (this.blockY <= 0) this.blockY = 0;

      // let x = item[0] + this.blockX;
      // let y = item[1] + this.blockY;
      // console.log(x);
      // if(x >= this.blockAll[0].length - 1) this.blockX = this.blockAll[0].length - x
      // if(y >= this.blockAll.length - 1) this.blockY = this.blockAll.lengt
      if (this.blockX >= temp) this.blockX = temp;
      if (this.blockY >= temp2) this.blockY = temp2;
    },

    // 创建键盘事件函数
    keyboardMove () {
      window.addEventListener("keydown", this.blockKeyboard);
    },

    // 销毁键盘事件函数
    keyboardRemove () {
      window.removeEventListener("keydown", this.blockKeyboard);
    },

    // 判断行满消除行函数
    blockGnough () {
      let temp = this.blockAll.length - 1;
      while (temp >= 0)
      {
        if (this.blockAll[temp].every(item => item === 1))
        {
          this.blockAll.splice(temp, 1);
          this.blockAll.unshift(new Array(this.blockAll[0].length).fill(0));
          // this.blockY++;
          this.numberRows++;
        }
        else temp--;
      };

      if (this.gameOverIs()) clearInterval(this.blockTimer)
      // if (!this.blockIs()) this.blockMove();
      // else this.gameOver = true;
    },

    // 判断当前下落方块是否接触到之前下落后的方块
    // dx表示x轴判断，dy表示y轴判断
    blockIs (dx = 0, dy = 0) {
      let temp = true;
      const temp2 = this.blockIsIs();
      //  console.log(temp2);
      this.currentBlock[this.currentBlockG].forEach(item => {
        const x = item[0] + this.blockX + dx;
        const y = item[1] + this.blockY + dy;
        let temp5 = true;
        // console.log(x !== item[0]);
        // console.log(String(item));
        for (let loop = 0; loop < temp2.length; loop++)
        {
          const temp3 = String(temp2[loop]);
          const temp4 = `${x},${y}`;
          if (temp3 === temp4)
          {
            temp5 = false;
            break;
          }
        }
        // console.log(this.blockY + this.blockYB,this.blockAll.length - 1);
        this.gameBorder();
        if (this.blockY + this.blockYB >= this.blockAll.length - 1 || (temp5 && this.blockAll[y][x] === 1)) temp = false;
        // console.log(temp);
      });
      return temp;
    },

    // 辅助下落判断的函数
    blockIsIs () {
      const temp = [];
      this.currentBlock[this.currentBlockG].forEach(item => {
        const x = item[0] + this.blockX;
        const y = item[1] + this.blockY;
        temp.push([x, y]);
      })
      return temp;
    },

    // 生成单独方块
    blockGenerate (is) {
      this.colorRandom = [Math.floor(Math.random() * 256), Math.floor(Math.random() * 256), Math.floor(Math.random() * 256)]
      this.currentBlockG = 0;
      this.blockX = 5;
      this.blockY = 0;
      this.difficultyKey = this.difficulty;
      this.keyboardRemove();
      this.blockRandom(is);
      this.blockMove();
      this.blockDown();
      this.keyboardMove();
    },

    // 方块下落函数
    blockDown () {
      clearInterval(this.blockTimer);
      this.blockTimer = setInterval(() => {
        // this.blockY--;
        if (this.blockY + this.blockYB >= this.blockAll.length - 1 || !this.blockIs(0, 1))
        {
          this.blockGnough();
          this.difficultyKey = this.difficulty;
          // console.log(this.difficulty);
          this.blockGenerate(1);
          this.blockRandom(3);
          this.blockMove(false);
          // this.blockMove();
        }
        else
        {
          this.blockClear();
          this.blockY++;
          this.blockMove();
        }
        // console.log(this.blockY, this.blockAll.length - 1);
        this.gameOverIs();
        // clearInterval(this.blockTimer)
      }, this.difficultyKey);
    },

    // 重置函数
    gameReset () {
      this.numberRows = 0;
      this.blockAll = [
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      ];
      this.keyboardRemove();
      clearInterval(this.blockTimer);
    },

    // 判断游戏结束函数
    gameOverIs () {
      if (this.blockY === 0 && !this.blockIs(0, 1)) 
      {
        this.keyboardRemove();
        clearInterval(this.blockTimer);
        setTimeout(() => {
          this.gameOver = true;
        }, 1000);
        return true;
      }
    },

    // 游戏总运行函数
    gameRun () {
      this.blockGenerate(2);
      this.blockRandom(3);
      this.blockMove(false);
      this.blockMove();
      this.blockDown();
      this.keyboardMove();
    },
  },

  mounted () {
    // 别删，我的臭宝
    console.log("%c此垃圾俄罗斯方块组件为华丽的小盒子所写，转载请注明出处", "background-color:rgb(30,30,30);border-radius:4px;font-size:12px;padding:4px;color:white;");
    window.addEventListener("keydown", e => {
      if (e.key === "Escape" && this.isGame) this.isGameStop();
    });
  }
}
</script>

<style lang="less" scoped>
// .colorWhite {
//   background: white !important;
// }

li {
  list-style: none;
}

#Block {
  width: 450px;
  height: 700px;
  position: fixed;
  right: 0;
  left: 0;
  top: 0;
  bottom: 0;
  margin: auto;

  .block-home {
    width: 100%;
    height: 100%;
    background: #ccc;
    display: flex;
    justify-content: space-around;
    align-items: center;
    flex-direction: column;

    p {
      font-size: 20px;
      font-weight: bold;
    }

    li {
      cursor: pointer;
      font-size: 15px;
      font-weight: bold;
      color: white;
      display: flex;
      justify-content: center;
      align-items: center;
      width: 80%;
      height: 40px;
      background: black;
      transition: all 0.3s;

      &:hover {
        color: black;
        background: white;
      }
    }
  }

  .block-game {
    width: 100%;
    height: 100%;
    background: #ccc;

    .block-game-head {
      width: 80%;
      height: 50px;
      display: flex;
      justify-content: space-around;
      align-items:center;

      .block-game-head-menu {
        width: 70px;
        height: 30px;
        background: black;
        color: white;
        font-weight: bold;
        display: flex;
        justify-content: center;
        align-items: center;
        border-radius: 5px;
        cursor: pointer;
        transition: all 0.3s;

        &:hover {
          color: black;
          background: white;
        }
      }

      div:nth-child(2)
      {
        font-size: 13px;
        font-weight: bold;
      }
    }

    .block-game-gameRegion {
      width: 75%;
      height: 90%;
      position: absolute;
      bottom: 1%;
      left: 2.5%;

      div {
        display: flex;
      }

      li {
        width: 30px;
        height: 30px;
      }
    }

    .block-game-nextRegion {
      height: 100%;
      width: 20%;
      position: absolute;
      right: 0;
      top: 0;
      background: black;
      display: flex;
      flex-direction: column;

      .block-game-nextRegion-next {
        width: 100%;
        height: 40%;
        background: rgb(141, 32, 32);
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;

        p {
          font-size: 13px;
          font-weight: bold;
          margin-bottom: 20px;
        }
        .block-game-nextRegion-next-all {
          width: 80px;
          height: 80px;
          display: flex;
          flex-wrap: wrap;
          background: black;

          div {
            width: 100%;
            height: 25%;
            display: flex;
          }

          li {
            width: 25%;
            height: 100%;
            // border-right: 1px solid black;
          }
        }
      }

      .block-game-nextRegion-scorePanel {
        flex: 1;
        background: rgb(24, 3, 3);
        display: flex;
        justify-content: center;
        align-items: center;

        p {
          font-size: 12px;
          color: white;
          font-weight: bold;
          display: flex;
          justify-content: center;
          align-items: center;
        }
      }
    }
  }

  .block-stop {
    user-select: none;
    position: absolute;
    top: 0;
    left: 0;
    background: rgba(0, 0, 0, 0.9);
    width: 100%;
    height: 100%;

    .block-stop-page {
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;

      div:nth-child(1) {
        margin-bottom: 50px;
      }

      div {
        font-size: 20px;
        background: white;
        width: 80%;
        height: 10%;
        display: flex;
        justify-content: center;
        align-items: center;
        font-weight: bold;
        cursor: pointer;
        transition: all 0.3s;

        &:hover {
          background: rgb(107, 67, 67);
          color: white;
        }
      }
    }
  }

  .block-gameOver {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    font-size: 30px;
    display: flex;
    justify-content: space-around;
    align-items: center;
    flex-direction: column;
    background: rgb(165, 129, 129);
    color: white;
    font-weight: bold;

    div {
      width: 80%;
      height: 15%;
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;

      div {
        width: 100%;
        height: 50px;
        margin-bottom: 10px;
        font-size: 25px;
        display: flex;
        justify-content: center;
        align-items: center;
        background: black;
        cursor: pointer;
        transition: all 0.3s;

        &:hover {
          color: black;
          background: white;
        }
      }
      p {
        font-size: 14px;
      }
    }
  }
}
</style>