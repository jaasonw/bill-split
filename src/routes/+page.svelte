<script lang="ts">
  import Footer from "$lib/components/footer.svelte";
  import Button from "$lib/components/ui/button/button.svelte";
  import * as Card from "$lib/components/ui/card";
  import { Input } from "$lib/components/ui/input";
  import { Label } from "$lib/components/ui/label";
  import * as Table from "$lib/components/ui/table";
  import { DataFrame } from "dataframe-js";
  import { Switch } from "$lib/components/ui/switch";
  import { Separator } from "$lib/components/ui/separator";
  // import LZString from "lz-string";
  // import { goto } from "$app/navigation";
  // import { page } from "$app/stores";

  interface Item {
    name: string;
    price: number;
    buyers: string[];
  }

  let items: Item[] = [];

  let people: string[] = [];

  // let htmlTable;
  let currentPerson = "";
  let currentItem = "";
  let currentPrice = 0;
  let tipInput: string = "";
  let taxInput: string = "";
  let tipAsProportion = true;
  let tip = 0;
  let tax = 0;

  // $: console.log(LZString.compressToBase64(JSON.stringify({"items": items, "people": people}))  );
  // $: {
  //   if (typeof window !== 'undefined') {
  //     const params = new URLSearchParams(window.location.search);
  //     params.set("data", LZString.compressToEncodedURIComponent(JSON.stringify({"items": items, "people": people})));
  //     goto(`?${params.toString()}`, { replaceState: true });
  //   }
  // }

  function createTable(
    items: Item[],
    people: string[],
    tip: number,
    tax: number,
    tipAsProportion: boolean,
  ) {
    let df = new DataFrame([["", ...people, "price"]]);
    Object.entries(items).map((item) => {
      let [id, { name, price, buyers }] = item;
      let row = [
        name,
        ...people.map((person) => {
          return { id: id, buyer: person };
        }),
        `$${price.toFixed(2)}`,
      ];
      df = df.push(row);
    });
    df = df.push([
      "subtotal",
      ...people.map((name) => getPartialSubtotal(name).toFixed(2)),
      `$${getSubtotal().toFixed(2)}`,
    ]);
    df = df.push([
      "tax",
      ...people.map((name) => getPartialAmount(name, tax).toFixed(2)),
      `$${tax.toFixed(2)}`,
    ]);
    df = df.push([
      "tip",
      ...(tipAsProportion
        ? people.map((name) => getPartialAmount(name, tip).toFixed(2))
        : people.map(() => (tip / people.length).toFixed(2))),
      `$${tip.toFixed(2)}`,
    ]);
    df = df.push([
      "total",
      ...people.map((name) => getPartialTotal(name).toFixed(2)),
      `$${getTotal().toFixed(2)}`,
    ]);
    // console.log(df.toArray());
    return df.toArray();
  }

  function getTotal() {
    return getSubtotal() + tip + tax;
  }

  function getPartialTotal(person: string) {
    let partialTip = tipAsProportion
      ? getPartialAmount(person, tip)
      : tip / people.length;

    return (
      getPartialSubtotal(person) + partialTip + getPartialAmount(person, tax)
    );
  }

  function validateTotals() {
    // sum all partial subtotals
    let calculated = people.reduce((acc, person) => {
      return acc + getPartialTotal(person);
    }, 0);
    // console.log(calculated, getTotal());
    return calculated === getTotal();
  }

  function getPartialPrice(item: number, person: string) {
    if (items[item].buyers.includes(person)) {
      return (items[item].price / items[item].buyers.length).toFixed(2);
    }
    return "0.00";
  }

  function getSubtotal() {
    return Object.entries(items).reduce((acc, [name, { price, buyers }]) => {
      return acc + price;
    }, 0);
  }

  function getPartialAmount(person: string, total: number) {
    const subtotal = getSubtotal();
    if (subtotal === 0) return 0;
    const percentage = total / getSubtotal();
    return getPartialSubtotal(person) * percentage;
  }

  function getPartialSubtotal(person: string) {
    return Object.entries(items).reduce((acc, [name, { price, buyers }]) => {
      if (buyers.includes(person)) {
        return acc + price / buyers.length;
      }
      return acc;
    }, 0);
  }

  function toggleBuyer(item: number, person: string) {
    if (items[item].buyers.includes(person)) {
      items[item].buyers = items[item].buyers.filter((i: any) => i !== person);
    } else {
      items[item].buyers = [...items[item].buyers, person];
    }
  }
  function addPerson() {
    people = [...people, currentPerson];
    currentPerson = "";
  }
  function addItem() {
    items = [
      ...items,
      {
        name: currentItem,
        price: Number(currentPrice),
        buyers: [],
      },
    ];
    currentItem = "";
    currentPrice = 0;
  }

  $: tip = Number(tipInput);
  $: tax = Number(taxInput);
  $: table = createTable(items, people, tip, tax, tipAsProportion);
</script>

<div class="flex flex-col items-center justify-center min-h-screen">
  <Card.Root class="w-6xl">
    <Card.Header>
      <Card.Title>bill splitter</Card.Title>
      <Card.Description>created by jasonw</Card.Description>
    </Card.Header>
    <Card.Content>
      {#if items.length !== 0 || people.length !== 0}
        <Table.Root>
          <Table.Header>
            <Table.Row>
              <Table.Head class="w-[100px]">Item</Table.Head>
              {#each people as person, j (j)}
                <Table.Head>{person}</Table.Head>
              {/each}
              <Table.Head class="text-right">Price</Table.Head>
            </Table.Row>
          </Table.Header>
          <Table.Body>
            {#each table as row, i (i)}
              {#if i !== 0}
                <Table.Row>
                  {#each row as item, j (j)}
                    {#if item.id}
                      <Table.Cell class="text-center p-0">
                        <Button
                          variant="ghost"
                          class="h-full w-full"
                          on:click={() => toggleBuyer(item.id, item.buyer)}
                          >{getPartialPrice(item.id, item.buyer) != "0.00"
                            ? `$${getPartialPrice(item.id, item.buyer)}`
                            : "-"}</Button
                        >
                      </Table.Cell>

                      <!-- Highlight the price if theres no buyers for it -->
                    {:else if j == row.length - 1 && row[j - 1].id && items[row[j - 1].id].buyers.length == 0}
                      <Table.Cell class="text-center font-bold text-red-500"
                        >{item}</Table.Cell
                      >
                      <!-- higlight the last row -->
                    {:else if i == table.length - 1 && !validateTotals()}
                      <Table.Cell class="text-center font-bold text-red-500"
                        >{item}
                      </Table.Cell>
                    {:else}
                      <Table.Cell class="text-center font-bold"
                        >{item}</Table.Cell
                      >
                    {/if}
                  {/each}
                </Table.Row>
              {/if}
            {/each}
          </Table.Body>
        </Table.Root>
      {/if}
    </Card.Content>

    <Card.Footer class="flex flex-col w-full">
      <div class="flex flex-col w-full border rounded-md p-5 gap-5">
        <h3 class="w-full font-lg font-bold">Input some data to begin</h3>
        <form
          class=" w-full flex gap-2 flex-col justify-center"
          on:submit|preventDefault={null}
        >
          <Input class="w-full" bind:value={currentPerson} />
          <Button class="w-full" type="submit" on:click={addPerson}
            >add person</Button
          >
        </form>
        <Separator />
        <form class=" w-full flex gap-2 flex-col justify-center">
          <div class="flex">
            <Input class="w-full" bind:value={currentItem} />
            <Input class="w-28" bind:value={currentPrice} type="number" />
          </div>
          <Button class="w-full" type="submit" on:click={addItem}
            >add item</Button
          >
        </form>

        <div class="flex flex-col gap-1 w-full">
          <Label class="w-1/3">Tip paid</Label>
          <Input class="" bind:value={tipInput} type="number" />
        </div>
        <div class="flex flex-col w-full">
          <Label>Tax paid:</Label>
          <Input class="" bind:value={taxInput} type="number" />
        </div>
        <div class="flex w-full items-center justify-between">
          <!-- tip split settings -->
          {#if tipAsProportion}
            <div class="flex flex-col">
              <Label>Tip as percent of subtotal</Label>
              <Label class="text-gray-400"
                >Everyone tips proportional to their meal</Label
              >
            </div>
          {:else}
            <div class="flex flex-col">
              <Label>Tip split evenly</Label>
              <Label class="text-gray-400">Everyone tips the same</Label>
            </div>
          {/if}
          <Switch class="" bind:checked={tipAsProportion} />
        </div>
        <!-- tip split settings -->
        <!-- <div class="flex w-full items-center justify-between">
          <div class="flex flex-col">
            <Label>Tip as</Label>
            {#if tipAsPercent}
              <Label class="text-gray-400"
                >Percent</Label
              >
            {:else}
              <Label class="text-gray-400"
                >Dollar value</Label
              >
            {/if}
          </div>
          <Switch class="" bind:checked={tipAsPercent} />
        </div> -->
      </div>
      <Footer />
    </Card.Footer>
  </Card.Root>
</div>
