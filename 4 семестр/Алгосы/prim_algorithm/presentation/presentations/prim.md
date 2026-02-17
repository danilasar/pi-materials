# Алгоритм Прима
Данила Григорьев

---

## Граф Чайник
Разбираемся в понятиях

--

- **Граф** &mdash; это структура из непустово множества вершин V, соединённых множеством рёбер E
- **Дерево** &mdash; связный граф, не содержащий циклов

--

- **Остов** &mdash; костяк предмета
- **Остовное дерево** &mdash; подграф с тем же числом вершин, что у исходного графа

--

- **Минимальный остов** &mdash; остовное дерево с минимальным числом рёбер

---

## Прим и примус: как починять?
Общая идея алгоритма Прима

--

## Идея

_По лемме о безопасном ребре мы можем строить минимальный остов постепенно, добавляя по одному ребру, в минимальности которого мы уверены_

1. Похитить алгоритм Дейкстры
<!-- .element: class="fragment fade-left" data-fragment-index="0" -->
2. Измерять веса вместо расстояний
<!-- .element: class="fragment fade-left" data-fragment-index="1" -->
3. Назвать это blazingly fast Prim's algorithm
<!-- .element: class="fragment fade-left" data-fragment-index="2" -->


--

### Раз уж blazingly fast

```rust
use std::cmp::Reverse;
use std::collections::{BTreeMap, BinaryHeap};
use std::ops::Add;

type Graph<V, E> = BTreeMap<V, BTreeMap<V, E>>;

fn add_edge<V: Ord + Copy, E: Ord + Add + Copy>(graph: &mut Graph<V, E>, v1: V, v2: V, c: E) {
    graph.entry(v1).or_default().insert(v2, c);
    graph.entry(v2).or_default().insert(v1, c);
}

// selects a start and run the algorithm from it
pub fn prim<V: Ord + Copy + std::fmt::Debug, E: Ord + Add + Copy + std::fmt::Debug>(
    graph: &Graph<V, E>,
) -> Graph<V, E> {
    match graph.keys().next() {
        Some(v) => prim_with_start(graph, *v),
        None => BTreeMap::new(),
    }
}

// only works for a connected graph
// if the given graph is not connected it will return the MST of the connected subgraph
pub fn prim_with_start<V: Ord + Copy, E: Ord + Add + Copy>(
    graph: &Graph<V, E>,
    start: V,
) -> Graph<V, E> {
    // will contain the MST
    let mut mst: Graph<V, E> = Graph::new();
    // a priority queue based on a binary heap, used to get the cheapest edge
    // the elements are an edge: the cost, destination and source
    let mut prio = BinaryHeap::new();

    mst.insert(start, BTreeMap::new());

    for (v, c) in &graph[&start] {
        // the heap is a max heap, we have to use Reverse when adding to simulate a min heap
        prio.push(Reverse((*c, v, start)));
    }

    while let Some(Reverse((dist, t, prev))) = prio.pop() {
        // the destination of the edge has already been seen
        if mst.contains_key(t) {
            continue;
        }

        // the destination is a new vertex
        add_edge(&mut mst, prev, *t, dist);

        for (v, c) in &graph[t] {
            if !mst.contains_key(v) {
                prio.push(Reverse((*c, v, *t)));
            }
        }
    }

    mst
}
```

--

<div style="position: fixed; top: 0; left: 0; right: 0; bottom: 0; z-index: 10; transform: translate(0%, 20%) scale(1.4)">

```rust
use std::cmp::Reverse;
use std::collections::{BTreeMap, BinaryHeap};
use std::ops::Add;

type Graph<V, E> = BTreeMap<V, BTreeMap<V, E>>;

fn add_edge<V: Ord + Copy, E: Ord + Add + Copy>(graph: &mut Graph<V, E>, v1: V, v2: V, c: E) {
    graph.entry(v1).or_default().insert(v2, c);
    graph.entry(v2).or_default().insert(v1, c);
}

// selects a start and run the algorithm from it
pub fn prim<V: Ord + Copy + std::fmt::Debug, E: Ord + Add + Copy + std::fmt::Debug>(
    graph: &Graph<V, E>,
) -> Graph<V, E> {
    match graph.keys().next() {
        Some(v) => prim_with_start(graph, *v),
        None => BTreeMap::new(),
    }
}

// only works for a connected graph
// if the given graph is not connected it will return the MST of the connected subgraph
pub fn prim_with_start<V: Ord + Copy, E: Ord + Add + Copy>(
    graph: &Graph<V, E>,
    start: V,
) -> Graph<V, E> {
    // will contain the MST
    let mut mst: Graph<V, E> = Graph::new();
    // a priority queue based on a binary heap, used to get the cheapest edge
    // the elements are an edge: the cost, destination and source
    let mut prio = BinaryHeap::new();

    mst.insert(start, BTreeMap::new());

    for (v, c) in &graph[&start] {
        // the heap is a max heap, we have to use Reverse when adding to simulate a min heap
        prio.push(Reverse((*c, v, start)));
    }

    while let Some(Reverse((dist, t, prev))) = prio.pop() {
        // the destination of the edge has already been seen
        if mst.contains_key(t) {
            continue;
        }

        // the destination is a new vertex
        add_edge(&mut mst, prev, *t, dist);

        for (v, c) in &graph[t] {
            if !mst.contains_key(v) {
                prio.push(Reverse((*c, v, *t)));
            }
        }
    }

    mst
}
```

</div>

---

## Пачом бананы?
Асимптотические оценки алгоритма

--

## Какой ты сегодня blazingly fast?

<style>
table { font-family: Jost; }
td, th {padding: 1rem !important; font-size: 1.5rem}
</style>
<table style="font-family: Jost;">
	<thead>
		<tr><th>Очередь</th><th>Асимптотика</th></tr>
	</thead>
	<tbody>
		<tr>
			<td>Наивная реализация</td>
			<td>
$O(V^2 + E)$
			</td>
		</tr>
		<tr>
			<td>
Двоичная куча
			</td>
			<td>
$O(E log V)$
			</td>
		</tr>
		<tr>
			<td>
Фиббоначиева куча
			</td>
			<td>
	$O(V log V + E)$
			</td>
		</tr>
	</tbody>
</table>

---

<style>
	.tablitsa, .tablitsa tr, .tablitsa td, .tablitsa th { border: 0 !important;  }
</style>

## Что вообще происходит
Алгоритм Прима на примере

--
<!-- .slide: data-auto-animate -->

<div style="display:flex; flex-direction: horizontal">
	<img src="assets/Mst_prima_1.png" />
	<table class="tablitsa">
		<thead><tr><th>a</th><th>b</th><th>c</th><th>d</th><th>e</th></tr></thead>
		<tbody><tr><td>0</td><td>$\infty$</td><td>$\infty$</td><td>$\infty$</td><td>$\infty$</td></tr></tbody>
	</table>
</div>

--
<!-- .slide: data-auto-animate -->

<div style="display:flex; flex-direction: horizontal">
	<img src="assets/Mst_prima_2.png" />
	<table class="tablitsa">
		<thead><tr><th>a</th><th>b</th><th>c</th><th>d</th><th>e</th></tr></thead>
		<tbody><tr><td>0</td><td>3</td><td>4</td><td>$\infty$</td><td>1</td></tr></tbody>
	</table>
</div>

--
<!-- .slide: data-auto-animate -->

<div style="display:flex; flex-direction: horizontal">
	<img src="assets/Mst_prima_3.png" />
	<table class="tablitsa">
		<thead><tr><th>a</th><th>b</th><th>c</th><th>d</th><th>e</th></tr></thead>
		<tbody><tr><td>0</td><td>3</td><td>4</td><td>7</td><td>1</td></tr></tbody>
	</table>
</div>

--
<!-- .slide: data-auto-animate -->

<div style="display:flex; flex-direction: horizontal">
	<img src="assets/Mst_prima_4.png" />
	<table class="tablitsa">
		<thead><tr><th>a</th><th>b</th><th>c</th><th>d</th><th>e</th></tr></thead>
		<tbody><tr><td>0</td><td>3</td><td>4</td><td>2</td><td>1</td></tr></tbody>
	</table>
</div>


---

# Спасибо. Вопросы?
