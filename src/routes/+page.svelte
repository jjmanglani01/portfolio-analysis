<script lang="ts">
  import Papa from 'papaparse';

  let bOldData = false;
  let bNewData = false;
  let holdingsData: any = {};

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
</script>
<div>
    <div>
        <span>Input old holding</span>
        <input type="file" placeholder="Enter old holding" on:change={onChange.bind(null, "old")} />
    </div>
    <div>
        <span>Input new holding</span>
        <input type="file" placeholder="Enter new holding" on:change={onChange.bind(null, "new")} />
    </div>
    {#if bOldData && bNewData}
      <table>
        {#each Object.values(holdingsData) as holding}
          <tr>
            <td>{holding["Symbol"]}</td>
            <td>{holding["Change"]}</td>
            <td>{holding["Change Perct"]}</td>
          </tr>
        {/each}
      </table>
    {/if}
</div>
