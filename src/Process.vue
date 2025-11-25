<script setup lang="ts">
import { i } from 'vite/dist/node/chunks/moduleRunnerTransport';
import { ref } from 'vue'

const cost_processstone_per_one = 100_000_000; // 研磨石1個あたりのコスト(メル)
const formatter = new Intl.NumberFormat('ja-JP', { style: 'decimal', minimumIntegerDigits: 1, useGrouping: true });

const targetProcessLevel = ref(10)
const discount = ref('None')

const processLevelList = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
type processResult = "Success" | "Failure" | "Destroy";
const processResultList: processResult[] = ["Success", "Failure", "Destroy"];

const process_probability_table_proto: {[key: number]: {[key in processResult]: number}} = {
    1:  { "Success": 100, "Failure":  0, "Destroy":  0 },
    2:  { "Success": 40,  "Failure": 60, "Destroy":  0 },
    3:  { "Success": 40,  "Failure": 55, "Destroy":  5 },
    4:  { "Success": 30,  "Failure": 60, "Destroy": 10 },
    5:  { "Success": 30,  "Failure": 60, "Destroy": 10 },
    6:  { "Success": 80,  "Failure":  0, "Destroy": 20 },
    7:  { "Success": 40,  "Failure": 40, "Destroy": 20 },
    8:  { "Success": 40,  "Failure": 40, "Destroy": 20 },
    9:  { "Success": 30,  "Failure": 40, "Destroy": 30 },
    10: { "Success": 30,  "Failure": 40, "Destroy": 30 },
}

const process_probability_table : {[key: number]: processResult[]} = {
  1: [],
  2: [],
  3: [],
  4: [],
  5: [],
  6: [],
  7: [],
  8: [],
  9: [],
  10: [],
}

processLevelList.forEach((level) => {
    processResultList.forEach((result) => {
        for(let count = 0; count < process_probability_table_proto[level][result]; count++)
        {
            process_probability_table[level].push(result);
        }
    });
});
console.log(process_probability_table);

const process_cost_table: {[key: number]: number} = {
    1: 1,
    2: 1,
    3: 1,
    4: 1,
    5: 1,
    6: 2,
    7: 2,
    8: 2,
    9: 3,
    10:3,
}

const getDiscount = (discountType: string) => {
  switch(discountType) {
    case 'None':
      return 1.0;
    case 'Guild':
      return 0.96;
    case 'Cadena':
      return 0.96;
    default:
      return 1.0;
  }
}

const process_result_content = ref({
  simulationCount: 0,
  consumeProcessstoneCount: 0,
  maxConsumeProcessstoneCount: 0,
  minConsumeProcessstoneCount: 0,
  averageConsumeProcessstoneCount: 0.0,
  discount: 'None'
})  

const process_calc = () => {
  console.log('-----Process Calc Start-----');
  process_result_content.value.simulationCount = 0;
  process_result_content.value.consumeProcessstoneCount = 0;
  const process_result_consumed_stones: number[] = [];

  // 100000回シミュレーション実行
  for(let count = 0; count < 100000; count++) {
    process_result_content.value.simulationCount++;
    let current_process_level = 1;
    let consume_processstone_count = 0;

    // console.log(`--- Simulation Count: ${count + 1} ---`);
    // 加工段階1～目標加工段階まで加工実行
    while(true){
        // 加工実行 テーブルから結果取得して加工石消費
        // console.log(current_process_level);
        // console.log(process_probability_table);
        // console.log(process_probability_table[current_process_level]);
        const result = process_probability_table[current_process_level][Math.floor(Math.random() * 100)];
        consume_processstone_count += process_cost_table[current_process_level];

        if(result === "Success") {
            if(current_process_level >= targetProcessLevel.value) {
                // 目標加工段階到達で離脱
                break;
            }
            // 成功:加工段階1段階アップ
            current_process_level++;
            
        }
        else if(result === "Failure") {
            // 失敗:加工段階1段階ダウン
            current_process_level--;
        }
        else if(result === "Destroy") {
            // 破壊:加工段階1に戻る
            current_process_level = 1;
        }
        // console.log(`Result: ${result}, Current Process Level: ${current_process_level}`);
    }
    // 目標加工段階到達時の消費加工石数を記録
    process_result_consumed_stones.push(consume_processstone_count);
  }
  console.log(process_result_consumed_stones);
  console.log('-----Process Calc End-----');

  // 最大消費加工石数を計算
  process_result_content.value.maxConsumeProcessstoneCount = Math.max(...process_result_consumed_stones);

  // 最小消費加工石数を計算
  process_result_content.value.minConsumeProcessstoneCount = Math.min(...process_result_consumed_stones);

  // 平均消費加工石数を計算
  process_result_content.value.averageConsumeProcessstoneCount = (process_result_consumed_stones.reduce((accumulator, currentValue) => accumulator + currentValue, 0) / process_result_content.value.simulationCount);
}
</script>

<template>
    <div id="process" class="row container-fluid">
        <h2>加工</h2>
        <div id="process_describe" class="card">
            宝玉を目標の段階になるまで加工することを10万回シミュレーションし、<br />
            加工石の消費数の最大・最小・平均とメルに換算した消費額を表示します。<br />
        </div>

        <div id="process_settings" class="card">
            <h3 class="card-title">設定</h3>
            <div class="field row">
                <div class="input-group">
                    <div class="card-subtitle col-3">目標加工段階</div>
                    <select id="targetProcess" class="form-select col" v-model.number="targetProcessLevel">
                        <option v-for="n in 10" :key="n" :value="n">{{ n }}</option>
                    </select>
                </div>
                
            </div>
            <div id="total" class="field row">
                <div class="input-group">
                    <div class="card-subtitle col-3">割引</div>
                    <select v-model="discount" class="form-select col">
                        <option value="None">なし</option>
                        <option value="Guild">商人(ギルドスキル, 4%割引)</option>
                        <option value="Cadena">交渉(カデナ, 4%割引)</option>
                    </select>
                </div>
            </div>
            <button id="button_process_calc" class="btn btn-primary mb-3" @click="process_calc">計算(時間がかかります)</button>
        </div>
        <div id="process_result" class="card">
            <h3>結果</h3>
             <table class="table table-bordered table-responsive">
                <thead>サマリー</thead>
                <tbody>
                    <tr><td>シミュレーション回数</td><td>{{ formatter.format(process_result_content.simulationCount) }} 回</td></tr>
                    <tr><td>最大消費加工石</td><td>{{ formatter.format(process_result_content.maxConsumeProcessstoneCount) }} 個</td></tr>
                    <tr><td>最小消費加工石</td><td>{{ formatter.format(process_result_content.minConsumeProcessstoneCount) }} 個</td></tr>
                    <tr><td>平均消費加工石</td><td>{{ formatter.format(process_result_content.averageConsumeProcessstoneCount) }} 個</td></tr>
                    <tr><td>　⇒メル<img src="./meso.png" alt="メル" class="inline" /></td><td>{{ formatter.format((process_result_content.averageConsumeProcessstoneCount * cost_processstone_per_one * getDiscount(process_result_content.discount)))}} メル</td></tr>
                </tbody>
            </table> 
        </div>
    </div>
</template>