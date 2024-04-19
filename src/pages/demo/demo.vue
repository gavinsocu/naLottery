<template>
    <div class="main-container">
        <h1>头号玩家抽奖程序</h1>
        <vs-select v-model="value" filter placeholder="Filter">
            <vs-option label="一等奖" value="1">一等奖</vs-option>
            <vs-option label="二等奖" value="2">二等奖</vs-option>
            <vs-option label="三等奖" value="3">三等奖</vs-option>
        </vs-select>
        <vs-button size="xl" @click="handle">
            开奖！
            <Icon type="choujian" />
        </vs-button>
        <vs-dialog v-model="active" not-close>
            <template #header>
                <h4 class="not-margin">恭喜获得</h4>
            </template>
            <div class="qian">
                <p class="ppp">{{ output }}</p>
            </div>
        </vs-dialog>
    </div>
</template>

<script>
import Icon from '@/component/icon.vue';
import cjplay from '@/assets/cj.mp3';

export default {
    components: {
        Icon
    },
    data() {
        return {
            output: '',
            active: false,
            value: '请选择奖项',
            winner: null,
            audioSrc: cjplay,
            // 每个奖项的概率设置，格式为 [起始值, 终止值, 累积概率]
            probabilities: [
                [1, 20, 60], // 1-20元的概率为60%
                [21, 40, 80], // 21-40元的概率为20%，累积概率为60% + 10% = 80%
                [41, 60, 90], // 41-60元的概率为10%，累积概率为60% + 10% = 90%
                [61, 80, 97], // 61-80元的概率为7%，累积概率为90% + 7% = 97%
                [81, 100, 100] // 81-100元的概率为3%，累积概率为97% + 3% = 100%
            ]
        };
    },
    methods: {
        drawWinner() {
            if (this.value == '3') {
                this.probabilities[0][2] = 20;
                this.probabilities[1][2] = 70;
            }
            const randomNumber = Math.random() * 100;

            // 根据随机数确定中奖区间
            let winningIndex = -1;
            for (let i = 0; i < this.probabilities.length; i++) {
                const [start, end, cumulativeProbability] = this.probabilities[i];
                if (randomNumber <= cumulativeProbability) {
                    winningIndex = i;
                    break;
                }
            }

            // 根据中奖区间确定中奖金额
            let prize = 0;
            if (winningIndex !== -1) {
                const [start, end] = this.probabilities[winningIndex];
                prize = Math.floor(Math.random() * (end - start + 1)) + start;
            }
            if (this.value == '1') prize = Math.round((prize * 130) / 100 + 198);
            else if (this.value == '2') prize = prize + 98;
            else if (this.value == '3')
                if (prize < 6) prize = 6;
                else if (prize > 98) prize = 98;
                else prize = prize;

            this.winner = prize + ' 元 ';
            this.active = true;
        },
        startRandomization() {
            this.audio = new Audio(this.audioSrc);
            this.audio.play();
            // 计数器，记录变化次数
            let count = 0;

            // 设置定时器，每隔一定时间改变 p 标签的值
            const interval = setInterval(() => {
                // 生成随机数
                const randomValue = Math.random() * 100; // 这里假设变化范围是 0 到 100
                // 更新 p 标签的值
                if (this.value == '1') this.output = randomValue.toFixed(0) * 1 + 198 + '元';
                else if (this.value == '2')
                    this.output = randomValue.toFixed(0) * 1 + 98 + '元'; // 保留两位小数
                else this.output = randomValue.toFixed(0) + '元';
                // 更新计数器
                count++;
                this.audio.addEventListener('ended', () => {
                    // 在音乐停止时执行的逻辑
                    // 清除定时器
                    clearInterval(interval);
                    this.output = this.winner;
                    this.disabled = false;
                    this.audio.pause();
                    this.audio.currentTime = 0;
                });
                // 如果变化次数达到 100 次，停止定时器，并将 p 标签的值设为目标数据，启用按钮
                // if (count === 36) {
                //     clearInterval(interval);
                //     this.output = this.winner;
                //     this.disabled = false;
                //     this.audio.pause();
                //     this.audio.currentTime = 0;
                // }
            }, 50); // 每隔 50 毫秒执行一次
        },
        handle() {
            if (this.value == '请选择奖项') return;
            this.drawWinner();
            this.startRandomization();
        }
    }
};
</script>

<style scoped>
.qian {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 500px;
    width: 100%;
    text-align: center;
}
.qian p {
    font-size: 100px;
    color: rgb(8, 8, 8);
    font-weight: bold;
}
h1 {
    font-weight: bolder;
}
</style>
