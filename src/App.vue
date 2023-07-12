<template>
    <div class="side">
        <div v-for="(item, index) in side_items" :key="index"
            :style="{ color: item.color, backgroundColor: index == selectSubject ? 'rgba(0, 255, 10, 1)' : 'rgba(0, 0, 0, 0)' }"
            @click="sideClickEvent(index)">
            {{ item.text }}
        </div>
    </div>
    <div class="dialog" v-show="selectSubject != -1">
        <div v-for="v, k, i in subject_name" :key="k" @click="dialogClickEvent(i, k)">
            {{ k }}
        </div>
    </div>
    <div class="main" ref="main">
        <div class="child" v-for="(item, index) in main_items" :key="index"
            :style="{ transition: MAIN_FONTSIZE_TRANSITION, color: item.color }" :id="item.id" :content="item.after">
            {{ item.text }}
        </div>
    </div>
    <div class="counter">
        <div class="text">
            <div> {{ dayCountdown }} </div>天
        </div>
    </div>
</template>
<script setup>
import { reactive, ref } from 'vue'

const MAIN_FONTSIZE_TRANSITION = 'font-size 0.5s ease-in-out, color 0.5s ease-in-out, width 0.5s ease-in-out'
const MAIN_SELECT_COLOR = 'rgba(255,255,0,1)'
const MAIN_BEFORE_COLOR = 'rgba(160,160,160,0.7)'
const MAIN_AFTER_COLOR = 'rgba(255,255,255,1)'
const SIDE_SELECT_COLOR = 'rgba(255,255,255,1)'
const SIDE_BEFORE_COLOR = 'rgba(190,190,190,1)'
const SIDE_AFTER_COLOR = 'rgba(255,255,255,1)'
const EXAM_DATE = '2024-6-7'


let daily_classes = [
    ['英', '数', '语', '走', '自', '化', '物', '自', '班'], //周日
    ['物', '英', '化', '语', '走', '体', '自', '数', '数'], //周一
    ['英', '数', '物', '语', '自', '化', '自', '语', '走'], //周二 
    ['化', '英', '数', '英', '地', '语', '自', '自', '自'], //周三
    ['化', '物', '数', '走', '自', '体', '语', '英', '自'], //周四
    ['英', '数', '语', '走', '自', '化', '物', '自', '班'], //周五
    ['英', '数', '语', '走', '自', '化', '物', '自', '班'], //周六 
]

const subject_name = {
    '自': '自习',
    '物': '物理',
    '英': '英语',
    '化': '化学',
    '语': '语文',
    '走': '走班',
    '生': '生物',
    '地': '地理',
    '历': '历史',
    '政': '政治',
    '体': '体育',
    '数': '数学',
    '班': '班会'
}

const common_schedule = {
    '00:00-07:39': '早自习',
    '07:40-08:19': 0,
    '08:20-08:29': '课间',
    '08:30-09:09': 1,
    '09:10-09:19': '课间',
    '09:20-09:59': 2,
    '10:00-10:29': '大课间',
    '10:30-11:09': 3,
    '11:10-11:59': '午休',
    '12:00-12:19': '午自习',
    '12:20-12:59': 4,
    '13:00-13:09': '课间',
    '13:10-13:49': 5,
    '13:50-13:59': '课间',
    '14:00-14:39': 6,
    '14:40-14:49': '课间',
    '14:50-15:29': 7,
    '15:30-15:39': '课间',
    '15:40-16:09': 8,
    '16:10-16:19': '课间',
    '16:20-17:19': '晚课',
    '17:20-18:29': '晚休',
    '18:30-20:39': '晚自习',
    '20:40-23:59': '放学',
}

let dayCountdown = '---'

let main_items = []

let side_items = []

let dialogDisplay = ref(false)

let selectSubject = ref(-1)

const main = ref(null)

function changeMain(index) {
    const element = main.value.children[index]
    for (let i = 0; i < main_items.length; i++) {
        if (i === index) {
            main_items[i].color = MAIN_SELECT_COLOR
            main_items[i].id = 'highlight'
            setTimeout(() => {
                element.scrollIntoView({ behavior: "smooth", block: "center", inline: "center" })
            }, 500);
        } else {
            main_items[i].id = ''
            if (i < index) {
                main_items[i].color = MAIN_BEFORE_COLOR
            } else {
                main_items[i].color = MAIN_AFTER_COLOR
            }
        }
    }
}

function changeSide(index) {
    for (let i = 0; i < side_items.length; i++) {
        if (i === index) {
            side_items[i].color = SIDE_SELECT_COLOR
        } else {
            if (i < index) {
                side_items[i].color = SIDE_BEFORE_COLOR
            } else {
                side_items[i].color = SIDE_AFTER_COLOR
            }
        }
    }
}

function updateDayCountdown() {
    let date = new Date()
    dayCountdown = Math.ceil(Math.abs(new Date(EXAM_DATE) - date) / (1000 * 60 * 60 * 24))
}

function updateMainOriginText() {
    let date = new Date()
    let week = date.getDay()
    let _main_items = new Array(common_schedule.length)
    Object.entries(common_schedule).sort((a, b) => a[0] - b[0]).forEach(function ([time, subject], i) {
        let text = subject
        if (typeof (text) == 'number') {
            text = subject_name[daily_classes[week][subject]] + '课'
        }
        _main_items[i] = { text: text, color: MAIN_AFTER_COLOR, id: '' }
    });
    main_items = reactive(_main_items)
}

function updateSideOriginText() {
    let date = new Date()
    let week = date.getDay()
    let _side_items = new Array(daily_classes[week].length)
    for (let i = 0; i < daily_classes[week].length; i++) {
        const subject = daily_classes[week][i];
        _side_items[i] = { text: subject, color: SIDE_AFTER_COLOR }
    }
    side_items = reactive(_side_items)
}

function getNowTimeStr() {
    const now = new Date();
    const hours = now.getHours().toString().padStart(2, '0');
    const minutes = now.getMinutes().toString().padStart(2, '0');
    const timeString = `${hours}:${minutes}`;
    for (const interval in common_schedule) {
        const [start, end] = interval.split('-');
        if (timeString >= start && timeString <= end) {
            return interval;
        }
    }
    return null;
}

function tick() {
    let date = new Date()
    let keys = Object.keys(common_schedule);
    let time = getNowTimeStr()
    keys.sort(function (a, b) {
        return a.localeCompare(b);
    });
    const index = keys.indexOf(getNowTimeStr())
    changeMain(index)
    let count = 0;
    for (let key in common_schedule) {
        if (typeof common_schedule[key] === 'number' && key.split('-')[1] <= time.split('-')[0]) {
            count++;
        }
    }
    changeSide(count)
    let remainingTime = Math.floor((new Date(new Date().toLocaleDateString('zh-CN') + ' ' + time.split('-')[1]) - date) / 1000) + 60;
    remainingTime = `${Math.floor(remainingTime / 60).toString().padStart(2, '0')}:${(remainingTime % 60).toString().padStart(2, '0')}`
    for (let i = 0; i < main_items.length; i++) {
        main_items[i].after = ''
    }
    main_items[index].after = remainingTime
}

function sideClickEvent(index) {
    console.log(index);
    console.log(dialogDisplay);
    
    if (selectSubject == index){
        selectSubject = -1
    }else{
        selectSubject = index
    }
}

function dialogClickEvent(index, subject){
    let date = new Date()
    let week = date.getDay()
    daily_classes[week][index] = subject
    side_items[selectSubject].text = subject
    Object.entries(common_schedule).sort((a, b) => a[0] - b[0]).forEach(function ([time, _subject], i) {
        if (_subject === selectSubject) {
            main_items[i].text = subject_name[subject] + '课' 
        }
    })
    selectSubject = -1
}

updateDayCountdown()
updateMainOriginText()
updateSideOriginText()
console.log(getNowTimeStr());
setInterval(tick, 1000)

</script>