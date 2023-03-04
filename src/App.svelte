<script>
    import { onMount, afterUpdate } from 'svelte';
    import Uplot from "./uplot_v3.svelte";
    var data;
    var last_ts = 0;
    var data_ready = false;
    // let ids = [100];
    let ids = [100, 101, 102, 103, 104, 105, 106, 107, 108, 109];
    onMount(async () => {
        async function readSensor(id, start_ts=0) {

            // let url = `http://132.163.53.82:3200/database/log.db/data?id=${id}`;
            let url = `http://132.163.53.82:3200/database/log.db/data?id=${id}&start=${start_ts}`;
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
            let sensor_data = await readSensor(id, start_ts);
            data.push( sensor_data.map(x=>x[2]) );
        }
        // console.log(data);
        data_ready = true;

        last_ts = data[0][data[0].length-1];
        console.log(last_ts);

    });

    async function append() {
        if (last_ts>0) {
            // let ids = [100, 108];
            // console.log('append, last_ts: ', last_ts);
            let new_ts = 0;
            let new_data = [];
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
                    new_data.push(json[0][2]);
                }
            }
            if (new_ts>0) last_ts = new_ts;
            console.log('new_data', new_data);

            for(var i=0; i<new_data.length; i++) {
                data[i].push(new_data[i])
            }
            data=data;
            // console.log('data size', data.length, data[0].length);

        } 
    }
    setInterval( append, 10000);

</script>

            <body>
<style>

    .uplot-text {
        margin-left: 150px;
        font-size: 18px;
    }
</style>
<div class="uplot-text">
    {#if data_ready }
        <Uplot data={data}/>
    {:else}
        Loading old data  
    {/if}
</div>
            </body>
