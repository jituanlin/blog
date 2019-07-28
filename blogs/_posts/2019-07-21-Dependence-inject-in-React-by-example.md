## Dependence inject in React by example

### Outline
Dependence inject is a common design pattern in OOP programming, \
although React community prefer functional programming, but OOP is
not evil, in some case, the dependence inject will be a strong tool
to decouple logic and provide safe way to reuse code.

In this blog, we will anatomy a case of using of dependence inject,
and concluded as follow:

1. Dependence inject let a `Component`(not only React component but
 also include all independent functional units), dependent `interface`
 rather than `implementation`. Because of this,

2. Dependence inject provide a relative safe way(interface constraint)
to extend `provider end` API (eg. component) for `custom end`(user of component),
rather than modify `provider end` code directly.

3. Dependence inject naturally split logic to difference component,
and then compose them in `control module`(the component which dependent this
component). Such that, the complexity be divide and keep at a relatively low
level.

### The example
Suppose we need to design a `Goods` component.
The `Goods` component include the purchase interactive behavior,
and choose time of receipt is one of step of purchase interaction.

So, the fist version we may be write same code as bellow:

```typescript
const Goods = (props: Props) => {
  // ...
  return (
    <div>
      // ...
      <ReceiptTimePicker /*...pass the props*/ />
      // ...
    </div>
  );
};

const GoodsPage = () => {
  const goodsInfo = useAsync(async () => {
    return fetchGoodsInfo();
  }, []);
  return <Goods goodsInfo={goodsInfo.value} />;
};
```

It looks nice, just render `ReceiptTimePicker` in Goods component,
and pass the props something like date constraint to it.

However, different kind of goods maybe has different receiving
method and different receipt time choose logic. So, there is a
goods need the user choose three times to receive goods in batches.

So, it is very different from the original `ReceiptTimePicker` logic,
then we write change the above code like this:

```typescript
enum ReceiptMethod {
  InBatches
}

const Goods = (props: Props) => {
  // ...
  return (
    <div>
      // ...
      {props.goodsInfo.receiptMethod === ReceiptMethod.InBatches ? (
        <InBatchesReceiptTimePicker /*...pass the props*/ />
      ) : (
        <ReceiptTimePicker /*...pass the props*/ />
      )}
      // ...
    </div>
  );
};

const GoodsPage = () => {
  const goodsInfo = useAsync(async () => {
    return fetchGoodsInfo();
  }, []);

  return <Goods goodsInfo={goodsInfo.value} />;
};
```

It looks good, but is it really like this?

I don't think so.

It looks not bad, because above code just a simple
example, but in real world, if we organize code like this,
we will soon get into trouble:

1. In real world, we need handle extra logic for
`InBatchesReceiptTimePicker` or `ReceiptTimePicker` in `Goods`
component, like props computed, synchronize state with them,
handle the event from them, etc...

2. Then, actually, we need to maintain two separate sets of
logic in `Goods` component by condition.

3. However, the `InBatchesReceiptTimePicker` will not be
the last one component we need to handle
for receipt time pick logic of different kind
of goods.

4. With the iteration of the business, finally,
all time pick logic will be entangled.


So, we refactor code like follow:

```typescript
enum ReceiptMethod {
  InBatches
}

type ReceiveTime = Date | ReadonlyArray<Date>;

interface ReceiptTimePicker {
  (onSelectTime: (time: ReceiveTime) => void): React.ReactNode;
}

const FallbackReceiptTimePicker: ReceiptTimePicker = (
  onSelectTime: (time: ReceiveTime) => void
) => {
  // ...
};

const InBatchesReceiptTimePicker: ReceiptTimePicker = (
  onSelectTime: (time: ReceiveTime) => void
) => {
  // ...
};

const getTimePickerByGoodsInfo = goodsInfo => {
  if (goodsInfo.receiptMethod === ReceiptMethod.InBatches) {
    return InBatchesReceiptTimePicker;
  }
  return FallbackReceiptTimePicker;
};

const Goods = (props: Props) => {
  const receiptTimePicker: ReceiptTimePicker = getTimePickerByGoodsInfo(
    props.goodsInfo
  );
  // ...
  return (
    <div>
      // ...
      <ReceiptTimePicker onSelectTime={/*...*/} />
      // ...
    </div>
  );
};

const GoodsPage = () => {
  const goodsInfo = useAsync(async () => {
    return fetchGoodsInfo();
  }, []);

  return <Goods goodsInfo={goodsInfo.value} />;
};
```

In this version
1. We add a new interface `ReceiptTimePicker` to
constraint all kind of time pick component, and just expose the
`onSelectTime` prop to interact with `control component`(`Goods` component).
Thus, all time pick logic will be limited in concrete implementation of
`ReceiptTimePicker`.

2. The `Goods` not directly dependent any concrete implementation of time
pick component, but the interface. Thus, if we need add a new kind of
time pick component, we just need create a new component which implement
the `ReceiptTimePicker` interface rather than modify the `Goods` component
directly (expandability).

3. We inject the `receiptTimePicker` to `Goods` by `getTimePickerByGoodsInfo`.
So we can config which implementation of `ReceiptTimePicker` to be inject.
Although this way is not normal dependence inject method,
but accord to definition, it is. We will discuss that lately.


### Conclusion of the example

It hard for me to take a example which simple enough to
show dependence inject pattern without introduce too
may business logic. Because, the dependence inject
is using for handle complex logic(in use case of this blog),
when the sample's logic is already sample enough, maybe introduce
dependence inject is not necessary.

Honestly, the example is not enough persuasiveness to
explain the necessity of introduce the dependence inject.
There are many design to achieve the same purpose.
However, the value of any tool is not depending on
whether there are other substitutes.
I think this example is good enough to bring us to the door.

The nature of dependence inject is:
1. Extra the common logic of different component, then

2. And inject it to the `control component` rather than
dependent it directly. Thus,

3. We could config be behavior by configuration(`function`, `xml`, `json` etc..) rather than
modify code directly.

### The inject way

There are many way to inject the dependence, so many framework provide
that.

But, framework is not required, we often see it although we may not realize
it. Like inject by props:

```typescript
    const add = (a, b) => a + b
```

Yes, this is dependence inject, we inject the dependence `a` and `b`
by props.

Like factory function:

```typescript
  let config: Promise<Config> | null = null
  export const resolveConfig = async(): Promise<Config> => {
      if (config === null) {
          config = fetchConfig()
      }
      return config
  }
```

Just satisfy dependence inject nature, the way is not be limited.




































