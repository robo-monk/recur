<script lang="ts">
  import { onMount } from "svelte";
  import { readable } from "svelte/store";
  import {
    Scene,
    PerspectiveCamera,
    WebGLRenderer,
    BoxGeometry,
    MeshBasicMaterial,
    Mesh,
    Camera,
    Material,
    Points,
    MathUtils,
    BufferGeometry,
    Float32BufferAttribute,
    PointsMaterial,
    Line,
    LineBasicMaterial,
  } from "three";

  let graphContainer: HTMLElement;
  let count: number = 0;
  const increment = () => {
    count += 1;
  };

  interface Coordinates {
    x?: number;
    y?: number;
    z?: number;
  }
  class Three {
    renderer: WebGLRenderer;
    scene: Scene;
    camera: Camera;

    constructor() {
      this.renderer = new WebGLRenderer();
      this.scene = new Scene();
      this.createCamera();

      // this.setSize()
      // this.renderer = new WebGLRenderer(WebGLRendererOptions);
    }
    createCamera(...options) {
      this.camera = new PerspectiveCamera(...options);
    }

    setCameraPosition({ x = null, y = null, z = null }: Coordinates): this {
      // im puking at this code duplication but probably the most efficient way to do it
      if (x != null) this.camera.position.x = x;
      if (y != null) this.camera.position.y = y;
      if (z != null) this.camera.position.z = z;

      return this;
    }

    get domElement() {
      return this.renderer.domElement;
    }

    setSize(width: number, height: number): this {
      // window.innerWidth, window.innerHeight
      this.renderer.setSize(width, height);
      this.createCamera(75, 1, width / height);
      return this;
    }

    appendTo(element: HTMLElement): this {
      element.appendChild(this.domElement);
      return this;
    }

    render(): this {
      this.renderer.render(this.scene, this.camera);
      return this;
    }
    animate(once: boolean = false): this {
      if (!once) requestAnimationFrame(() => this.animate());
      this.render();
      return this;
    }

    addPoint({ x = null, y = null, z = null }: Coordinates = {}): this {
      const vertices = [];

      // const x = MathUtils.randFloatSpread(2000);
      // const y = MathUtils.randFloatSpread(2000);
      // const z = MathUtils.randFloatSpread(2000);
      vertices.push(x || 0, y || 0, -150);

      return this.addVertices(vertices);
    }

    addVertices(vertices) {
      const geometry = new BufferGeometry();
      geometry.setAttribute(
        "position",
        new Float32BufferAttribute(vertices, 3)
      );

      const material = new PointsMaterial({ color: "#fff", size: 0.5 });

      const points = new Points(geometry, material);

      const lineGeo = new BufferGeometry().setFromPoints(vertices);
      const lineMaterial = new LineBasicMaterial({ color: "#ffffff" });
      const line = new Line(geometry, lineMaterial);

      this.scene.add(points);
      // this.scene.add(line);
      return this;
    }

    addMesh({
      geometry = new BoxGeometry(),
      material = new MeshBasicMaterial(),
      mesh = null,
    }: // mesh = new Mesh(geometry, material),
    {
      geometry?: BoxGeometry;
      material?: Material;
      mesh?: Mesh;
    }): this {
      if (!mesh) {
        mesh = new Mesh(geometry, material);
      }
      // const geometry = new BoxGeometry();
      // const material = new MeshBasicMaterial({ color: 0x00ff00 });
      // const cube = new Mesh(geometry, material);
      this.scene.add(mesh);
      return this;
    }
  }

  (async function makeThree() {
    await new Promise((r) => {
      onMount(() => r(null));
    });

    const size = [2500, 1500];
    const three = new Three()
      .setSize(size[0], size[1])
      .appendTo(graphContainer)
      // .addMesh({ material: new MeshBasicMaterial({ color: "red" }) })
      .setCameraPosition({ z: 300 })
      .render();

    function plot(
      fn: (i: number) => number,
      min = 0,
      max = 100,
      resolution = 10
    ): any[] {
      let vertices = [];
      let points = []
      // for (let i: number = min; i++; i <= max) {
      // let i = 1
      let i = min;
      while (i <= max) {
        // console.log(i, fn(i))
        // resolution *= i+1
        let point = [i / 20 - 100, fn(i) * 200];
        points.push(point);
        vertices.push(...point, 1);
        i += 1 / resolution;
      }
      // console.log('adding vertices', vertices)
      // vertices.push(i+1, fn(i+1), -1)
      // three.addPoint({
      //   x: i,
      //   y: fn(i),
      // });
      // }
      console.log(points.length);
      three.addVertices(vertices);
      return points;
    }

    function xn(x, r) {
      return r * x * (1 - x);
    }

    function x100(x, r) {
      let i = 0;
      while (i < (Math.random() + 1) * r * 500) {
        x = xn(x, r);
        // if (res.has(x)) return Math.random() > .5 ? res[res.size - 1] : x
        // if (res.has(x)) return x
        i++;
        // yield Math.abs(ans) > 100 ? 100 : ans
      }

      return x;
    }
    // function xn(x, r) {
    //   return r*x*(1-x)
    // }

    function* newR() {
      let r = 0.0;
      let ans = 0.53;
      while (true) {
        // ans = xn(ans, r);
        // ans = xn(ans, r);
        ans = x100(0.42, r);
        r += r > 1 ? (Math.log(r) - Math.log(r - 1)) / 5000 : 0.005;
        yield ans;
        // yield Math.abs(ans) > 100 ? 100 : ans
      }
    }

    const rr = newR();
    let points = plot(
      (i): number => {
        let value = rr.next().value || 1;
        return value;
        // rr.next().value || 1
      },
      2,
      5000,
      10
      // 10
    );

    console.log(points)
    // points.forEach(point => {
    //   point
    // })

    // let ii = 0;
    // while (ii< 1000) {
    //   console.log(rr.next().value)
    //   ii++
    // }

    three.render();
    // while (i <= 50) {
    //   newR()
    //   // ans = xn(ans, r)
    //   // plot(() => ans, 1, 50, 1);
    //   i++
    // }

    // console.log(xn(10, 10));
    // newR()
    // newR()
    // newR()
    // newR()
    // newR()
    // newR()
    // newR()
    // newR()
    // setInterval(newR, 50);
    // three.addPoint();
  })();
  // const renderer = new WebGLRenderer();
  // renderer.setSize(window.innerWidth, window.innerHeight);
  // document.body.appendChild(renderer.domElement);

  // const geometry = new BoxGeometry();
  // const material = new MeshBasicMaterial({ color: 0x00ff00 });
  // const cube = new Mesh(geometry, material);
  // scene.add(cube);

  // function animate() {
  //   requestAnimationFrame(animate);
  //   renderer.render(scene, camera);
  // }
  // animate();
</script>

<div bind:this={graphContainer} />

<button on:click={increment}>
  Clicks: {count}
</button>

<style>
  button {
    font-family: inherit;
    font-size: inherit;
    padding: 1em 2em;
    color: #ff3e00;
    background-color: rgba(255, 62, 0, 0.1);
    border-radius: 2em;
    border: 2px solid rgba(255, 62, 0, 0);
    outline: none;
    width: 200px;
    font-variant-numeric: tabular-nums;
    cursor: pointer;
  }

  button:focus {
    border: 2px solid #ff3e00;
  }

  button:active {
    background-color: rgba(255, 62, 0, 0.2);
  }
</style>
