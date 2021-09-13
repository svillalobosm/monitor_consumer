<script>
  // export let data;
  import DataTable, {
    Head,
    Body,
    Row,
    Cell,
    Label,
    Pagination,
  } from "@smui/data-table";
  import Select, { Option } from "@smui/select";
  import IconButton from "@smui/icon-button";
  import LinearProgress from "@smui/linear-progress";

  let rowsPerPage = 10;
  let currentPage = 0;
  let items = [];
  let sort = "id";
  let sortDirection = "ascending";

  $: start = currentPage * rowsPerPage;
  $: end = Math.min(start + rowsPerPage, items.length);
  $: slice = items.slice(start, end);
  $: lastPage = Math.max(Math.ceil(items.length / rowsPerPage) - 1, 0);

  $: if (currentPage > lastPage) {
    currentPage = lastPage;
  }

  const getData = async () => {
    const url = "http://54.153.23.105:9595/v1/companies/production.json";
    const resp = await fetch(url);
    const data = resp.json();
    if (resp.ok) {
      return data;
    } else {
      throw new Error("Error to connect API");
    }
  };
  $: getData().then((data) => (items = data));

  function handleSort() {
    items.sort((a, b) => {
      const [aVal, bVal] = [a[sort], b[sort]][
        sortDirection === "ascending" ? "slice" : "reverse"
      ]();
      if (typeof aVal === "string") {
        return aVal.localeCompare(bVal);
      }
      return aVal - bVal;
    });
    items = items;
  }
</script>

<div class="dataTable">
  {#await getData()}
    <div class="linearProgress">
      <LinearProgress indeterminate />
    </div>
  {:then}
    <DataTable
      sortable
      bind:sort
      bind:sortDirection
      on:MDCDataTable:sorted={handleSort}
      table$aria-label="User list"
      style="width: 100%;"
    >
      <Head>
        <Row>
          <Cell style="width:10%;" numeric columnId="cpnId">
            <!-- For numeric columns, icon comes first. -->
            <IconButton class="material-icons">arrow_upward</IconButton>
            <Label>CPN</Label>
          </Cell>
          <Cell style="width:10%; text-align:center;" columnId="cpnCode">
            <Label>RUT</Label>
            <!-- For non-numeric columns, icon comes second. -->
            <IconButton class="material-icons">arrow_upward</IconButton>
          </Cell>
          <Cell style="width:80%;" columnId="cpnName">
            <Label>NOMBRE</Label>
            <IconButton class="material-icons">arrow_upward</IconButton>
          </Cell>
        </Row>
      </Head>
      <Body>
        {#each slice as item (item.cpnId)}
          <Row>
            <Cell style="width:10%;" numeric>{parseInt(item.cpnId)}</Cell>
            <Cell style="width:10%; text-align:center;">{item.cpnCode}</Cell>
            <Cell style="width:80%;">{item.cpnName.toUpperCase().trim()}</Cell>
          </Row>
        {/each}
      </Body>
      <Pagination slot="paginate">
        <svelte:fragment slot="rowsPerPage">
          <Label>Filas por página</Label>
          <Select variant="outlined" bind:value={rowsPerPage} noLabel>
            <Option value={10}>10</Option>
            <Option value={25}>25</Option>
            <Option value={100}>100</Option>
          </Select>
        </svelte:fragment>
        <svelte:fragment slot="total">
          {start + 1}-{end} de {items.length}
        </svelte:fragment>

        <IconButton
          class="material-icons"
          action="first-page"
          title="First page"
          on:click={() => (currentPage = 0)}
          disabled={currentPage === 0}>first_page</IconButton
        >
        <IconButton
          class="material-icons"
          action="prev-page"
          title="Prev page"
          on:click={() => currentPage--}
          disabled={currentPage === 0}>chevron_left</IconButton
        >
        <IconButton
          class="material-icons"
          action="next-page"
          title="Next page"
          on:click={() => currentPage++}
          disabled={currentPage === lastPage}>chevron_right</IconButton
        >
        <IconButton
          class="material-icons"
          action="last-page"
          title="Last page"
          on:click={() => (currentPage = lastPage)}
          disabled={currentPage === lastPage}>last_page</IconButton
        >
      </Pagination>
    </DataTable>
  {/await}
</div>

<style>
  .dataTable {
    margin: 3rem 4rem 2rem 4rem;
  }
  .linearProgress {
    margin-top: 30%;
  }
</style>
