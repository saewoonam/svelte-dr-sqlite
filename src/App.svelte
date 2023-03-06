<script>
    import { onMount, afterUpdate } from 'svelte';
    import Uplot from "./uplot_v3.svelte";
    import {linear} from 'everpolate';
    var data;
    var loading_id = -1;
    var last_ts = 0;
    var data_ready = false;
    let host = '132.163.53.82:3200';
    // let ids = [100];
    let ids = [100, 101, 102, 103, 104, 105, 106, 107, 108, 109];
    let DC2018;
    onMount(async () => {
        console.log('everpolate linear', linear);
        let diode_list_url = `http://${host}/database/log.db/diode_list`;
        let calibration_url_prefix = `http://${host}/database/calibration.db/`;
        var list = await fetch(diode_list_url);
        list = await list.json();
        console.log('diode_list', list);
        let cal_name = 'DC2018'
        var list = await fetch(`${calibration_url_prefix}${cal_name}`);
        list = await list.json();
        let T = list['data'].map(x=>x[0]);
        let V = list['data'].map(x=>x[1]);
        DC2018 = (x) => {return linear(x, V, T).map((x)=> (x>0) ? x : null)}
        console.log(cal_name, DC2018(0.5));

        async function readSensor(id, start_ts=0) {
            // let url = `http://132.163.53.82:3200/database/log.db/data?id=${id}`;
            let url = `http://${host}/database/log.db/data?id=${id}&start=${start_ts}`;
            // let url = `http://127.0.0.1:3200/database/log.db/data?id=${id}&start=${start_ts}`;
            var response = await fetch(url); 
            var json = await response.json(); 
            return json['data']
        }
        let start_ts = 0; // Date.now()/1000 - 100; 
        console.log('start_ts', start_ts);
        let sensor_data = await readSensor(100, start_ts);
        data = [
            sensor_data.map(x=>x[0])
        ];
        for (const id of ids) { 
            loading_id = id;
            let sensor_data = await readSensor(id, start_ts);
            data.push( DC2018(sensor_data.map(x=>x[2])) );
        }
        // console.log(data);
        data_ready = true;

        last_ts = data[0][data[0].length-1];
        console.log(last_ts);

    });
    var table_data = [];
    async function append() {
        if (last_ts>0) {
            // let ids = [100, 108];
            // console.log('append, last_ts: ', last_ts);
            let new_ts = 0;
            let new_data = [];
            // new_data = [];
            for (const id of ids) {
                // fetch all data since last_ts that was plotted
                let url = `http://132.163.53.82:3200/database/log.db/data?id=${id}&start=${last_ts+1}`;
                // console.log('url', url);
                var json = await fetch(url).then(response => response.json()).then(res=>res['data']);
                // console.log(id, json, last_ts);

                // only add one timestamp of data even if there is more
                if ((json.length>0) & (new_ts==0)) {
                    new_ts = json[0][0];
                    console.log('new_ts to add', json[0][0]);
                    new_data.push(new_ts);
                }
                if (json.length>0) {
                    new_data.push(DC2018(json[0][2]));
                }
            }
            if (new_ts>0) last_ts = new_ts;
            // console.log('new_data', new_data);

            for(var i=0; i<new_data.length; i++) {
                data[i].push(new_data[i])
            }
            data=data;
            // table_data = new_data; 
            table_data = new_data.map(
                (x,i)=>  (i>0) ? x[0].toFixed(2) : (new Date(x.toFixed(0)*1000)).toLocaleString()
            );
            console.log('table_data', table_data); 
            // console.log('data size', data.length, data[0].length);

        } 
    }
    setInterval( append, 10000);

</script>

<style>
    .sidebar {
        height: 200px;
        width: 200px;
        position: fixed;
        top: 0;
        left: 0;
        padding-top: 100px;
        /* background-color: lightblue; */
    }

    .sidebar div {
        padding: 8px;
        font-size: 24px;
        display: block;
    }
    .uplot-text {
        margin-left: 200px;
        font-size: 18px;
    }
    table {
        border: 1px solid;
    }
    th, td {
        padding: 5px;
        text-align: center;
        border: 1px solid;
    }
</style>
<body>
    <!--
    <h1> {last_ts} {table_data[1]} </h1>
    -->
    <div class="sidebar">
        {#if table_data.length>0}
            {table_data[0]}
        {/if}
        <table>
            <tr>
                <th>ID </th>
                <th>reading</th>
            </tr>
            {#each table_data as reading,i}
                {#if i>0}
                    <tr>
                        <td> {ids[i-1]} </td>
                        <td> {reading} </td>
                    </tr>
                {/if}
            {/each}
        </table>
    </div>
    <div class="uplot-text">
        {#if data_ready }
            <Uplot data={data} labels={ids}/>
        {:else}
            Loading old data, id {loading_id} 
        {/if}
    </div>
</body>
