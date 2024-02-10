<script lang="ts">
  import Footer from "$lib/components/footer.svelte";
  import Button from "$lib/components/ui/button/button.svelte";
  import * as Card from "$lib/components/ui/card";
  import { Input } from "$lib/components/ui/input";
  import { Label } from "$lib/components/ui/label";

  let total = 0.00;
  let subtotal = 0;
  let tip = 0;
  let tax = 0;
  $: tax = parseFloat(((total - subtotal) / subtotal).toFixed(2));
</script>

<div class="flex flex-col items-center justify-center h-screen">
  <Card.Root class="w-6xl">
    <Card.Header>
      <Card.Title>tab splitter</Card.Title>
      <Card.Description>created by jasonw</Card.Description>
    </Card.Header>
    <Card.Content>
      <form id="form">
        <div class="grid w-full items-center gap-4">
          <div class="flex flex-col space-y-1.5">
            <Label for="name">subtotal</Label>
            <Input
              name="subtotal"
              type="number"
              step="0.01"
              placeholder=""
              min="0"
              bind:value={subtotal}
            />
            <Label for="name">total</Label>
            <Input
              name="total"
              type="number"
              step="0.01"
              placeholder=""
              min="0"
              bind:value={total}
            />
            <Label for="name">tip %</Label>
            <Input
              name="tip"
              type="number"
              step="0.01"
              placeholder=""
              min="0"
              bind:value={tip}
            />
          </div>
        </div>
      </form>
      <div class="flex flex-col">
        calculated tax: {tax}
        tip without tax: {(subtotal + (subtotal * tip)).toFixed(2)}
      </div>
    </Card.Content>
    <Card.Footer class="flex flex-col w-full">
      <div class=" w-full flex justify-center">
        <Button class="w-full" form="form" type="submit">submit</Button>
      </div>
      <Footer />
    </Card.Footer>
  </Card.Root>
</div>
