<style>

</style>

<svelte:head>
  <title>Sapper Unit Compare</title>
</svelte:head>

<h2 class="text-center">Unit Compare</h2>
<br />

<div class="container">
  <form on:submit|preventDefault={addRow}>
    <div class="row">
      <div class="col">
        <label>Name</label>
        <input type="text" name="name" bind:value={name}/>
      </div>
      <div class="col">
        <label>Price</label>
        <input type="text" name="price" bind:value={price}/>
      </div>
      <div class="col">
        <label>Unit Count</label>
        <input type="text" name="unitcount" bind:value={unitcount}/>
      </div>
      <div class="col">
        <label>Unit</label>
        &nbsp;
        <select bind:value={unitkey} on:change="{getTargetValue}">
          {#each units as unit}
            <option value={unit.key}>
              {unit.key}
            </option>
          {/each}
        </select>
      </div>
      <div class="col">
        <button class="btn btn-primary">&plus;&nbsp;Add</button>
      </div>
    </div>
  </form>
</div>

{#if err}
<div class="alert alert-danger">{err}</div>
{/if}

<br />

<table class="table table-sm">
  <thead>
    <tr>
      <td>
        Item
      </td>
      <td>
        Price
      </td>
      <td class="text-right">
        per Unit
      </td>
      <td class="text-right">
        per LB
      </td>
      <td>
      </td>
    </tr>
  </thead>
  <tbody>
    {#each rows as row, i}
    <tr>
      <td>
        {row.name}
      </td>
      <td>
        ${row.price}
        <span class="text-muted">for</span>
        {row.unitcount} {row.unitkey}
      </td>
      <td class="text-right">
        ${toUnitPriceFixed(row)}
      </td>
      <td class="text-right">
        ${toUnitPriceInLbsFixed(row)}
      </td>
      <td class="text-center">
        <button class="btn btn-danger btn-sm" on:click="{() => delRow(row)}">&times;</button>
      </td>
    </tr>
    {/each}
  </tbody>
</table>

<script>
  import { onMount } from 'svelte';
  import get from 'lodash/get';
  import find from 'lodash/find';
  let price, name, unitcount, unitkey, rows, err;
  const lskey = 'uc_rows';
  const FIXED = 3;
  const units = [
    { key: 'lb', lbMult: 1, },
    { key: 'oz', lbMult: (1/16), },
  ];
  const ls = typeof window !== 'undefined' ? window.localStorage : null;
  rows = [];

  onMount(() => {
    console.log('onMount');
    if (ls && window.localStorage.getItem(lskey)) {
      rows = JSON.parse(ls.getItem(lskey));
    }
    if (rows && rows.length === 0) {
      rows.push({
        name: 'Chicken Breast',
        unitcount: 1,
        price: 1.99,
        unitkey: 'lb',
        createdAt: Date.now(),
      });

      // rows.push({
      //   name: 'Chicken Breast',
      //   unitcount: 32,
      //   price: 1.99,
      //   unitkey: 'lb',
      //   createdAt: Date.now(),
      // });

      // rows.push({
      //   name: 'Chicken Breast',
      //   unitcount: 32,
      //   price: 8.99,
      //   unitkey: 'lb',
      //   createdAt: Date.now(),
      // });

      // rows.push({
      //   name: 'Tofu',
      //   unitcount: 32,
      //   price: 8.99,
      //   unitkey: 'oz',
      //   createdAt: Date.now(),
      // });
    }
  });

  function toUnitPrice(row) {
    return row.price / row.unitcount;
  }

  function toUnitPriceFixed(row) {
    return toUnitPrice(row).toFixed(FIXED);
  }

  function toUnitPriceInLbsFixed(row) {
    const unit = find(units, u => row.unitkey === u.key);
    if (!unit) {
      return 'N/A';
    }
    return (toUnitPrice(row) / unit.lbMult).toFixed(FIXED);
  }

  function reset() {
    unitcount = '';
    name = '';
    price = '';
    // the default is oz, because the premise of this app
    // is that chicken is sold in lbs but tofu is sold in oz
    unitkey = 'oz';
  }

  function syncRows () {
    rows = rows;
    if (ls) {
      ls.setItem(lskey, JSON.stringify(rows));
    }
  }

  function addRow() {
    console.log('addRow');
    if (!(name && price && unitkey && unitcount)) {
      err = 'all values required';
      return;
    }
    if (!(isValidNumber(price) && isValidNumber(unitcount))) {
      err = 'price and unitcount must be valid numbers';
      return;
    }
    err = null;
    const row = {
      name,
      unitcount: parseFloat(unitcount),
      price: parseFloat(price),
      unitkey,
      createdAt: Date.now(),
    };
    rows.push(row);
    syncRows();
    reset();
  }
  function delRow(row) {
    const index = rows.indexOf(row);
    if (index >= 0) {
      rows.splice(index, 1);
      syncRows();
    }
  }
  function getTargetValue($event) {
    return get($event, 'target.value') || '';
  }
  function isValidNumber(value) {
    return !isNaN(parseFloat(value));
  }

  reset();
</script>
