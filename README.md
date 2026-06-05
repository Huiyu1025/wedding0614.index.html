<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>婚宴座位查詢系統</title>
    <style>
        body { font-family: Arial, sans-serif; padding: 20px; background-color: #f9f9f9; }
        .container { max-width: 600px; margin: 0 auto; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        h1 { text-align: center; color: #333; }
        button { background-color: #ff4d4d; color: white; border: none; padding: 10px 20px; border-radius: 4px; cursor: pointer; font-size: 16px; width: 100%; margin-bottom: 10px; }
        button:hover { background-color: #ff3333; }
        #output { margin-top: 20px; padding: 10px; background: #eee; border-radius: 4px; white-space: pre-wrap; font-family: monospace; max-height: 400px; overflow-y: auto; }
    </style>
</head>
<body>

<div class="container">
    <h1> wedding 0614 座位管理 </h1>
    <button id="btn-init">1. 初始化：將 30 桌座位資料寫入 Firebase</button>
    <button id="btn-read">2. 讀取：從 Firebase 撈出目前座位資料</button>
    
    <div id="output">點擊上方按鈕執行操作...</div>
</div>

<!-- 引入 Firebase SDK (使用全球 CDN) -->
<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
    import { getDatabase, ref, set, get, child } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

    // 您的 Firebase 核心配置，已代入您的資料庫網址
    const firebaseConfig = {
        databaseURL: "https://wedding0614-default-rtdb.asia-southeast1.firebasedatabase.app/"
    };

    // 初始化 Firebase
    const app = initializeApp(firebaseConfig);
    const db = getDatabase(app);

    // 您前面確認好的完整 30 桌座位名單資料
    const weddingSeatsData = {
        "table_01": { name: "主桌", capacity: 12, guests: ["陳加毫", "陳重憲", "楊琇雅", "李渤昇", "張簡秀香", "楊順興", "吳蕙妤", "吳鎮州", "陳美云", "陳延昌", "盧虹蘭", "吳鎮宇"] },
        "table_02": { name: "客桌", capacity: 10, guests: ["吳珍美", "吳珍美", "小姑婆", "小姑婆", "大姑婆", "大姑婆", "李采恩", "李采恩", "李采恩", "許廷光"] },
        "table_03": { name: "客桌", capacity: 10, guests: ["陳簡家政", "陳簡家政", "何秀冠", "陳煜崧", "劉智勇", "劉智勇", "簡明仁", "簡明仁", "周麗華"] },
        "table_04": { name: "客桌", capacity: 10, guests: ["陳忠吏", "陳忠吏", "陳忠吏", "陳俊雄", "王彩霞", "蘇美瑜", "郭鎮雄", "黃珮瀅", "黃偉展", "柳秀英"] },
        "table_05": { name: "客桌", capacity: 10, guests: ["鄭志龍", "許若涵", "張德仁", "Steven", "沈江", "沈江", "沈江", "何宜芳", "徐弘宇", "張簡維真"] },
        "table_06": { name: "客桌", capacity: 10, guests: ["白翠英", "白翠英", "許銘能", "許宥廷", "張簡少均", "張簡少均", "陳信安", "陳信安", "潘坤宗", "潘坤宗"] },
        "table_07": { name: "客桌", capacity: 10, guests: ["莊婉如", "張顯爵", "張以理", "沈育德", "鄭佳玲", "鄭佳玲", "鄭旭棠", "鄭旭棠", "鄭旭棠", "鄭旭棠"] },
        "table_08": { name: "客桌", capacity: 10, guests: ["王敏龍", "郭永泰", "楊鴻振", "梁緒傑", "楊麗君", "謝德榮", "林永茂", "王麗靖", "周宏偉", "楊仲修"] },
        "table_09": { name: "客桌", capacity: 10, guests: ["吳美珍", "蘇瓊密", "蘇瓊密", "傅紹杰", "李家妮", "王春福", "王春福", "林欣欣", "林欣欣", "采瑩"] },
        "table_10": { name: "客桌", capacity: 10, guests: ["羅文良", "陳瀅如", "羅丹琪", "陳羿均", "陳漢俊", "吳妮靜", "林冠廷", "陳瀅因"] },
        "table_11": { name: "客桌", capacity: 10, guests: ["楊欽富", "黃秋華", "陳樹村", "林佩樺", "吳順明", "蘇峾萱", "黃乾隆", "鄭寶貴", "陳珀貴", "葉珍芬"] },
        "table_12": { name: "客桌", capacity: 10, guests: ["李天從", "李天從", "蘇財國", "陳明珠", "谷寶華", "林錦崇", "黃金標", "王敏", "蔡志宏", "柯麗姫"] },
        "table_13": { name: "客桌", capacity: 10, guests: ["張天振", "李武宗", "鄭富仁", "黃懷德", "吳明璇", "程昶芬", "阮和豐", "林榮昌", "林裕仁"] },
        "table_14": { name: "客桌", capacity: 10, guests: ["吳昆哲", "傅宥豪", "陳旻揚", "陳旻揚", "陳奕霖", "王雨婕", "王雨婕", "胡雅璇", "王博誼", "王奕茗"] },
        "table_15": { name: "客桌", capacity: 10, guests: ["伏萱", "伏萱", "黃琦元", "李品儀", "李品儀", "李品儀", "李品儀", "陳映璇", "李季庭", "簡利霓"] },
        "table_16": { name: "客桌", capacity: 10, guests: ["Fanny", "許譯文", "蔡瑋軒", "周虹妤", "林禹葇", "林禹葇", "陳睿真", "余念欣", "郭祉晨"] },
        "table_17": { name: "客桌", capacity: 10, guests: ["陳麗琴", "陳麗琴", "陳麗琴", "陳麗鴻", "陳麗鴻", "陳麗鴻", "阿公", "阿嬤", "外勞"] },
        "table_18": { name: "客桌", capacity: 10, guests: ["黃華志", "蔡尚恒", "林建陽", "呂政偉", "莊鎮宣", "莊鎮宣", "張愛芬", "張程顥", "林文欽", "林志峰"] },
        "table_19": { name: "客桌", capacity: 10, guests: ["林旻盈", "Timmi", "林玟呤", "林玟呤", "張健偉", "張健偉", "詹佩佩", "Cubi", "蔡佳玲"] },
        "table_20": { name: "客桌", capacity: 10, guests: ["朱瑞麒", "朱瑞麒", "朱瑞麒", "李政憲", "李政憲", "李耀星", "李耀星", "李耀星"] },
        "table_21": { name: "客桌", capacity: 10, guests: ["楊粢婷", "楊粢婷", "周誠", "林琬淳", "林琬淳", "蔡品柔", "蘇育禾", "柯伶儒", "張佑誠", "葉馨"] },
        "table_22": { name: "客桌", capacity: 10, guests: ["李紹宏", "李Bobo", "李Bobo", "陳怡安", "陳怡安", "朱哲均", "朱哲均", "陳炭謂", "陳禹豪", "洪瑞呈"] },
        "table_23": { name: "客桌", capacity: 10, guests: ["賴柏桓", "鄭登仁", "鄭登仁", "張桐瑜", "程智裕", "吳欣盈", "吳欣盈", "李冠佑(Tony", "許凱翔", "蘇大爺"] },
        "table_24": { name: "客桌", capacity: 10, guests: ["林根宏", "林根宏", "林根宏", "林根宏", "林根宏", "林根宏", "林根由", "林根葉", "林倚占"] },
        "table_25": { name: "客桌", capacity: 10, guests: ["薛庭雯", "張國基", "蕭逸錚", "呂英聖", "簡宏益", "周義翔", "王前婷", "林敬斌", "蔡康和", "蔡康和"] },
        "table_26": { name: "客桌", capacity: 10, guests: ["朱郁庭", "林小姐", "羅昇弘", "羅昇弘", "羅昇弘", "張永龍", "張永龍", "易統", "易統", "林青山"] },
        "table_27": { name: "客桌", capacity: 10, guests: ["黃義松", "黃義松", "黃義松", "黃義松", "林建宏", "林建宏", "陳美玲", "楊舒惠", "蕭逸錚", "蕭逸錚"] },
        "table_28": { name: "客桌", capacity: 10, guests: ["王瓊慧", "蘇裕勝", "蘇裕勝", "吳健全"] },
        "table_29": { name: "客桌", capacity: 10, guests: ["陳泓諺", "Wendy", "王乙縉", "王乙縉", "蘇義芳", "蘇義芳", "莊敏男", "莊敏男", "林冠宇", "林泓佐"] },
        "table_30": { name: "客桌", capacity: 10, guests: ["蔡乙瑄", "薛沛柔", "薛沛柔", "游凱文", "游凱文", "李依庭", "許育銘", "顏紹龍", "顏紹龍"] }
    };

    const outputDiv = document.getElementById("output");

    // 功能 1：點擊按鈕一鍵將名單寫入 Firebase 
    document.getElementById("btn-init").addEventListener("click", () => {
        outputDiv.innerText = "正在寫入資料到 Firebase...";
        set(ref(db, "wedding_seats/"), weddingSeatsData)
            .then(() => {
                outputDiv.innerText = "✅ 成功！30桌資料已完整寫入您的 Firebase Realtime Database！";
            })
            .catch((error) => {
                outputDiv.innerText = "❌ 失敗，錯誤原因: " + error.message;
            });
    });

    // 功能 2：點擊按鈕從 Firebase 撈取最新資料
    document.getElementById("btn-read").addEventListener("click", () => {
        outputDiv.innerText = "正在從 Firebase 讀取資料...";
        const dbRef = ref(getDatabase());
        get(child(dbRef, "wedding_seats/")).then((snapshot) => {
            if (snapshot.exists()) {
                outputDiv.innerText = JSON.stringify(snapshot.val(), null, 2);
            } else {
                outputDiv.innerText = "⚠️ 資料庫目前是空的，請先點擊按鈕 1 進行初始化。";
            }
        }).catch((error) => {
            outputDiv.innerText = "❌ 讀取失敗: " + error.message;
        });
    });
</script>

</body>
</html>
