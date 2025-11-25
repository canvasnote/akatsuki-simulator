<script setup lang="ts">
import { ref } from 'vue'

const formatter = new Intl.NumberFormat('ja-JP', { style: 'decimal', minimumIntegerDigits: 1, useGrouping: true });

const whet_table_proto = {
  // 研磨回数10回に対する各ステータス上昇確率
  // 情報元: https://home.gamer.com.tw/creationDetail.php?sn=6091116
  1: {
    1: {weight: 65, bonus: 25 },
    2: {weight:  5, bonus: 50 },
    3: {weight: 10, bonus: 75 },
    4: {weight: 20, bonus: 100},
  },
  2: {
    1: {weight: 65, bonus: 25 },
    2: {weight:  5, bonus: 50 },
    3: {weight: 10, bonus: 75 },
    4: {weight: 20, bonus: 100},
  },
  3: {
    1: {weight: 65, bonus: 25 },
    2: {weight:  5, bonus: 50 },
    3: {weight: 10, bonus: 75 },
    4: {weight: 20, bonus: 100},
  },
  4: {
    1: {weight: 70, bonus: 25 },
    2: {weight:  6, bonus: 50 },
    3: {weight:  9, bonus: 75 },
    4: {weight: 15, bonus: 100},
  },
  5: {
    1: {weight: 70, bonus: 25 },
    2: {weight:  6, bonus: 50 },
    3: {weight:  9, bonus: 75 },
    4: {weight: 15, bonus: 100},
  },
  6: {
    1: {weight: 75, bonus: 50  },
    2: {weight:  7, bonus: 100 },
    3: {weight:  8, bonus: 150},
    4: {weight: 10, bonus: 200 },
  },
  7: {
    1: {weight: 75, bonus: 50  },
    2: {weight:  7, bonus: 100 },
    3: {weight:  8, bonus: 150},
    4: {weight: 10, bonus: 200 },
  },
  8: {
    1: {weight: 80, bonus: 50  },
    2: {weight:  8, bonus: 100 },
    3: {weight:  7, bonus: 150 },
    4: {weight:  5, bonus: 200 },
  },
  9: {
    1: {weight: 80, bonus: 50  },
    2: {weight:  8, bonus: 100 },
    3: {weight:  7, bonus: 150 },
    4: {weight:  5, bonus: 200 },
  },
  10: {
    1: {weight: 85, bonus: 50  },
    2: {weight:  9, bonus: 100 },
    3: {weight:  4, bonus: 150 },
    4: {weight:  2, bonus: 200 },
  },
}

const whet_table: {[key: number]: number[]} = {
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

for(let level = 1; level <= 10; level++) {
  for(let itemno = 1; itemno <= 4; itemno++) {
    for(let count = 0; count < whet_table_proto[level][itemno].weight; count++)
      whet_table[level].push(whet_table_proto[level][itemno].bonus);
  }
  // console.log(whet_table[level])
}

const firstCount = ref(3)
const firstBonus = ref(200)
const totalBonus = ref(700)

const whet_result_content = ref({
  valueMax: 0,
  valueMin: 0,
  valueAvg: 0,
  simulationCount: 0,
  probability_first: 0,
  probability_total: 0,
  probability_both: 0,
  probability_either: 0,
  achievedCount: 0,
  consumeWhetstoneCountFirst: 0,
  consumeWhetstoneCountTotal: 0,  
  consumeWhetstoneCountBoth: 0,
  consumeWhetstoneCountEither: 0,
  averageConsumeWhetstoneCount: 0.0
})  

const whet_calc = () => {
  console.log('-----Whet Calc Start-----');
  whet_result_content.value.simulationCount = 0;
  const whet_result = [];
  const whet_success_first_bonus = new Set();
  const whet_success_total_bonus = new Set();

  // 10万回シミュレーション実行
  for(let count = 0; count < 100000; count++) {
    whet_result_content.value.simulationCount++;
    let current_bonus = 0;
    let first_x_bonus = 0;

    // レベル1～10まで研磨実行
    for(let level = 1; level <= 10; level++) {
      // ランダムに研磨結果を取得
      const bonus = whet_table[level][Math.floor(Math.random() * whet_table[level].length)];
      // ステータス上昇を累積
      current_bonus += bonus
      if (level <=firstCount.value) {
        first_x_bonus += bonus;
      }
    }
    // 研磨結果のステータスを保存
    whet_result.push(current_bonus);

    // 最初のN回の研磨でステータス+Xを達成したか判定
    if(first_x_bonus >= firstBonus.value){
      whet_success_first_bonus.add(count);
    }
    
    // 10回の合計でステータス+Xを達成したか判定
    if (current_bonus >= totalBonus.value) {
      whet_success_total_bonus.add(count);
    }

  }
  console.log(whet_result);
  console.log(whet_success_first_bonus);
  console.log(whet_success_total_bonus);
  console.log('-----Whet Calc End-----');
  whet_result_content.value.valueMax = Math.max(...whet_result);
  whet_result_content.value.valueMin = Math.min(...whet_result);
  whet_result_content.value.valueAvg = (whet_result.reduce((a, b) => a + b, 0) / whet_result.length);
  console.log('Max: ' + whet_result_content.value.valueMax);
  console.log('Min: ' + whet_result_content.value.valueMin);
  console.log('Avg: ' + whet_result_content.value.valueAvg);

  // 条件を満たす回数をカウント
  const success_count_first = whet_success_first_bonus.size;
  const success_count_total = whet_success_total_bonus.size;
  const success_count_both = whet_success_first_bonus.intersection(whet_success_total_bonus).size;
  const success_count_either = whet_success_first_bonus.union(whet_success_total_bonus).size;
  console.log('First Bonus Success Count: ' + success_count_first);
  console.log('Total Bonus Success Count: ' + success_count_total);
  console.log('Both Conditions Success Count: ' + success_count_both);
  console.log('Either Condition Success Count: ' + success_count_either);

  // 確率を計算
  whet_result_content.value.probability_first = (success_count_first / whet_result_content.value.simulationCount * 100);
  whet_result_content.value.probability_total = (success_count_total / whet_result_content.value.simulationCount * 100);
  whet_result_content.value.probability_both = (success_count_both / whet_result_content.value.simulationCount * 100);
  whet_result_content.value.probability_either = (success_count_either / whet_result_content.value.simulationCount * 100);
  console.log('success Probability(first): ' + whet_result_content.value.probability_first + ' %');
  console.log('success Probability(total): ' + whet_result_content.value.probability_total + ' %');
  console.log('success Probability(both): ' + whet_result_content.value.probability_both + ' %');
  console.log('success Probability(either): ' + whet_result_content.value.probability_either + ' %');

  // 平均消費研磨石数を計算
  whet_result_content.value.consumeWhetstoneCountFirst = 10 / (whet_result_content.value.probability_first / 100);
  whet_result_content.value.consumeWhetstoneCountTotal = 10 / (whet_result_content.value.probability_total / 100);
  whet_result_content.value.consumeWhetstoneCountBoth = 10 / (whet_result_content.value.probability_both / 100);
  whet_result_content.value.consumeWhetstoneCountEither = 10 / (whet_result_content.value.probability_either / 100);
}
</script>

<template>
    <div id="whet_settings" class="row container-fluid">
        <h2>研磨</h2>
        <div id="whet_describe" class="card">
            宝玉を10段階まで研磨することを10万回シミュレーションし、<br />
            条件①と②を満たす確率とそれに必要な研磨石数を計算します。<br />
        </div>

        <div id="whet_settings" class="card">
            <h3 class="card-title">設定</h3>
            <div id="first" class="field row">
                <div class="input-group">
                    <div class="card-subtitle col-2">条件①</div>
                    <label for="firstCount" class="input-group-text">最初の</label>
                    <select id="firstCount" class="form-select" v-model.number="firstCount">
                        <option v-for="n in 10" :key="n" :value="n">{{ n }}</option>
                    </select>
                    <div class="input-group-text">回の研磨でステータス+</div>
                    <input type="number" class="form-control" v-model.number="firstBonus" />
                </div>
            </div>

            <div id="total" class="field row">
                <div class="input-group">
                    <div class="card-subtitle col-2">条件②</div>
                    <label class="input-group-text">10回の合計でステータス+</label>
                    <input type="number" class="form-control" v-model.number="totalBonus" />
                </div>
            </div>

            <div id="whet_calc" class="field row">
                <button id="button_whet_calc" class="btn btn-primary" @click="whet_calc">計算</button>
            </div>
            
        </div>

        <div id="whet_result" class="card">
            <h3 class="card-title">結果</h3>
            <div id="whet_result_content">
                <!-- 結果表示エリア -->
                <table class="table table-bordered table-responsive">
                    <thead>サマリー</thead>
                    <tbody>
                    <tr><td>シミュレーション回数</td><td>{{ formatter.format(whet_result_content.simulationCount) }} 回</td></tr>
                    <tr><td>平均ステータス</td><td>{{ whet_result_content.valueAvg.toFixed(2) }}</td></tr>
                    <tr><td>最大ステータス</td><td>{{ formatter.format(whet_result_content.valueMax) }}</td></tr>
                    <tr><td>最小ステータス</td><td>{{ formatter.format(whet_result_content.valueMin) }}</td></tr>
                    </tbody>
                </table> 

                <table class="table table-bordered table-responsive">
                    <thead>条件①を満たす</thead>
                    <tbody>
                    <tr><td>条件を満たす確率</td><td>{{ whet_result_content.probability_first.toFixed(2) }} %</td></tr>
                    <tr><td>平均消費研磨石<img src="./whetstone.png" alt="研磨石" class="inline" /></td><td>{{ whet_result_content.consumeWhetstoneCountFirst.toFixed(2) }} 個</td></tr>
                    <tr><td>　⇒砕けた竜の鱗<img src="./dragonscale.png" alt="竜の鱗" class="inline" /></td><td>{{ (whet_result_content.consumeWhetstoneCountFirst * 10).toFixed(2) }}個分</td></tr>
                    <tr><td>　⇒竜の鱗包み<img src="./dragonscalepack.png" alt="竜の鱗包み" class="inline" /></td><td>{{ (whet_result_content.consumeWhetstoneCountFirst * 10 / 20).toFixed(2) }}個分</td></tr>
                    <tr><td>　⇒メイプルポイント換算</td><td>{{ formatter.format(whet_result_content.consumeWhetstoneCountFirst * 20) }} p分</td></tr>
                    </tbody>
                </table> 

                <table class="table table-bordered table-responsive">
                    <thead>条件②を満たす</thead>
                    <tbody>
                    <tr><td>条件を満たす確率</td><td>{{ whet_result_content.probability_total.toFixed(2) }} %</td></tr>
                    <tr><td>平均消費研磨石<img src="./whetstone.png" alt="研磨石" class="inline" /></td><td>{{ whet_result_content.consumeWhetstoneCountTotal.toFixed(2) }} 個</td></tr>
                    <tr><td>　⇒砕けた竜の鱗<img src="./dragonscale.png" alt="竜の鱗" class="inline" /></td><td>{{ (whet_result_content.consumeWhetstoneCountTotal * 10).toFixed(2) }}個分</td></tr>
                    <tr><td>　⇒竜の鱗包み<img src="./dragonscalepack.png" alt="竜の鱗包み" class="inline" /></td><td>{{ (whet_result_content.consumeWhetstoneCountTotal * 10 / 20).toFixed(2) }}個分</td></tr>
                    <tr><td>　⇒メイプルポイント換算</td><td>{{ formatter.format(whet_result_content.consumeWhetstoneCountTotal * 20) }} p分</td></tr>
                    </tbody>
                </table> 

                <table class="table table-bordered table-responsive">
                    <thead>両方を満たす</thead>
                    <tbody>
                    <tr><td>条件を満たす確率</td><td>{{ whet_result_content.probability_both.toFixed(2) }} %</td></tr>
                    <tr><td>平均消費研磨石<img src="./whetstone.png" alt="研磨石" class="inline" /></td><td>{{ whet_result_content.consumeWhetstoneCountBoth.toFixed(2) }} 個</td></tr>
                    <tr><td>　⇒砕けた竜の鱗<img src="./dragonscale.png" alt="竜の鱗" class="inline" /></td><td>{{ (whet_result_content.consumeWhetstoneCountBoth * 10).toFixed(2) }}個分</td></tr>
                    <tr><td>　⇒竜の鱗包み<img src="./dragonscalepack.png" alt="竜の鱗包み" class="inline" /></td><td>{{ (whet_result_content.consumeWhetstoneCountBoth * 10 / 20).toFixed(2) }}個分</td></tr>
                    <tr><td>　⇒メイプルポイント換算</td><td>{{ formatter.format(whet_result_content.consumeWhetstoneCountBoth * 20) }} p分</td></tr>
                    </tbody>
                </table> 

                <table class="table table-bordered table-responsive">
                    <thead>どちらか一方でも満たす</thead>
                    <tbody>
                    <tr><td>条件を満たす確率</td><td>{{ whet_result_content.probability_either.toFixed(2) }} %</td></tr>
                    <tr><td>平均消費研磨石<img src="./whetstone.png" alt="研磨石" class="inline" /></td><td>{{ whet_result_content.consumeWhetstoneCountEither.toFixed(2) }} 個</td></tr>
                    <tr><td>　⇒砕けた竜の鱗<img src="./dragonscale.png" alt="竜の鱗" class="inline" /></td><td>{{ (whet_result_content.consumeWhetstoneCountEither * 10).toFixed(2) }}個分</td></tr>
                    <tr><td>　⇒竜の鱗包み<img src="./dragonscalepack.png" alt="竜の鱗包み" class="inline" /></td><td>{{ (whet_result_content.consumeWhetstoneCountEither * 10 / 20).toFixed(2) }}個分</td></tr>
                    <tr><td>　⇒メイプルポイント換算</td><td>{{ formatter.format(whet_result_content.consumeWhetstoneCountEither * 20) }} p分</td></tr>
                    </tbody>
                </table> 
            </div>
        </div>
</div>
</template>