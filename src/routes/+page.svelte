<script lang="ts">
  import Papa from 'papaparse';
  import DataTable from 'datatables.net-dt';
  import { onMount } from 'svelte';

  let bOldData = false;
  let bNewData = false;
  let holdingsData: any = {};
  let table: any;

  function onChange(data: string, event: Event) {
    const element = event.target as HTMLInputElement;  
    const files = element.files;
    if (files && files[0]) {
      parseFile(files[0], data);
    }
  }

  function parseFile(file: File, data: string) {
    Papa.parse(file, {
	    header: true,
	    skipEmptyLines: true, //other option is 'greedy', meaning skip delimiters, quotes, and whitespace.
	    columns: null, //or array of strings,
      complete: function (results: any) {
        const holdings = results.data;
        if (data === "old") {
          bOldData = true;
          for (let holding of holdings) {
            holdingsData[holding["ISIN"]] = holding;
          }
        } else if (data === "new") {
          bNewData = true;
          for (let holding of holdings) {
            if (holdingsData[holding["ISIN"]]) {
              const oldHolding = holdingsData[holding["ISIN"]];
              const change = parseFloat(holding["Previous Closing Price"]) - parseFloat(oldHolding["Previous Closing Price"]);
              const changePerct = change * 100 / parseFloat(oldHolding["Previous Closing Price"]);
              holdingsData[holding["ISIN"]] = {...oldHolding, "Change": change, "Change Perct": changePerct };
            }
          }
        }
      },
	    error: function (error: any) {
        console.log("Error:", error);
      },
    });   
  }

  onMount(() => {
    table = new DataTable("#table-holding");
  });

  $: {
    if (bOldData && bNewData) {
      table.clear();
      Object.values(holdingsData).forEach((holding: any) => {
        if (holding["Change"] !== undefined) {
          table.row.add([holding["Symbol"], holding["Change"], holding["Change Perct"]]);
        }
      });
      table.draw();
    }
  }
</script>
<div class="p-4">
  <div class="grid grid-cols-2 gap-4 mb-4">
    <label class="form-control w-full max-w-xs">
      <div class="label">
        <span class="label-text">Input old holding</span>
      </div>
      <input type="file" class="file-input file-input-bordered w-full max-w-xs" placeholder="Enter old holding" on:change={onChange.bind(null, "old")} />
    </label>
    <label class="form-control w-full max-w-xs">
      <div class="label">
        <span class="label-text">Input new holding</span>
      </div>
      <input type="file" class="file-input file-input-bordered w-full max-w-xs" placeholder="Enter new holding" on:change={onChange.bind(null, "new")} />
    </label>
  </div>
  <table id="table-holding" class="display" style="width:100%">
    <thead>
      <tr>
        <th>Symbol</th>
        <th>Change</th>
        <th>Change Perct</th>
      </tr>
    </thead>
    <tbody />
  </table>
</div>
