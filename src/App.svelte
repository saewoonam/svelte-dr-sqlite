<script>
    import { onMount, afterUpdate } from 'svelte';
  import Uplot from "./uplot_v3.svelte";
    var data;
    var last_ts = 0;
    var data_ready = false;
    onMount(async () => {
        async function readSensor(id) {
            let url = `http://132.163.53.82:3200/database/log.db/data?id=${id}`;
            var response = await fetch(url); 
            var json = await response.json(); 
            return json['data']
        }
        let ids = [100, 101, 102, 103, 104, 105, 106, 107, 108, 109];
        let sensor_data = await readSensor(100);
        data = [
            sensor_data.map(x=>x[0])
        ];
        for (const id of ids) { 
            let sensor_data = await readSensor(id);
            data.push( sensor_data.map(x=>x[2]) );
        }
        console.log(data);
        data_ready = true;
        /*
        data = [
            sensor_data.map(x=>x[0]),
            sensor_data.map(x=>x[2]),
            sensor_data2.map(x=>x[2])
        ];
        */
        /*
        last_ts = data[0][data[0].length-1];
        console.log(last_ts);
        */
    });
    /* 
	data = [
			[1546300800, 1546387200],    // x-values (timestamps)
			[        35,         15],    // y-values (series 1)
			[        90,         15],    // y-values (series 2)
			];
	let count = 0;
    */
	async function append() {
        if (last_ts>0) {
            let ids = [100, 108];
            let new_ts = 0;
            let new_data = [];
            for (const id of ids) {
                let url = `http://132.163.53.82:3200/database/log.db/data?id=${id}&start=${last_ts+1}`;
                var json = await fetch(url).then(response => response.json()).then(res=>res['data']);
                console.log(id, json);
                if ((json.length>0) & (new_ts==0)) {
                    new_ts = json[0][0];
                    console.log(json[0][0]);
                    new_data.push(new_ts);
                }
                if (json.length>0) {
                    new_data.push(json[0][2]);
                }
            }
            if (new_ts>0) last_ts = new_ts;
            console.log('new_data', new_data);
            /* 
            for(var i=0; i<new_data.length; i++) {
                data[i].push(new_data[i])
            }
            data=data;
            */
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

{#if data_ready }
<div class="uplot-text">
<Uplot data={data}/>
</div>
{/if}
</body>
