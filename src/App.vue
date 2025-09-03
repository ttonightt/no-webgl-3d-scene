<script setup>
import { onMounted, useTemplateRef } from "vue";
const pi = Math.PI;
const pi2 = pi * 2;


const vec = (x, y, z) => ({ x, y, z });

const cross = (v1, v2) => ({
	x: (v1.y * v2.z) - (v1.z * v2.y),
	y: (v1.z * v2.x) - (v1.x * v2.z),
	z: (v1.x * v2.y) - (v1.y * v2.x)
});

const dot = (v1, v2) => (v1.x * v2.x) + (v1.y * v2.y) + (v1.z * v2.z);

const norm = v => {

	const mag = Math.sqrt( (v.x ** 2) + (v.y ** 2) + (v.z ** 2) );

	return {
		x: v.x / mag,
		y: v.y / mag,
		z: v.z / mag
	};
};

const diff = (v, v0) => ({
	x: v.x - v0.x,
	y: v.y - v0.y,
	z: v.z - v0.z
});

const lookAtM = (targetV, cameraV) => {

	const UP = vec(0, 1, 0);

	const L = norm(diff(targetV, cameraV));
	const s = norm(cross(L, UP));
	const u = cross(s, L);

	return [
		s.x, s.y, s.z, -dot(s, cameraV),
		u.x, u.y, u.z, -dot(u, cameraV),
		-L.x, -L.y, -L.z, dot(L, cameraV),
		0, 0, 0, 1
	];
};

const intrinsicM = (f, plainW, plainH) => [

	f, 0, plainW / 2,
	0, f, plainH / 2,
	0, 0, 1
];

const proj = (x0, y0, z0, target, camera, f) => {

	const E = lookAtM(target, camera);
	const I = intrinsicM(f, window.innerWidth, window.innerHeight);

	const V = [
		(E[0] * x0) + (E[1] * y0) +  (E[2] * z0) + E[3],
		(E[4] * x0) + (E[5] * y0) +  (E[6] * z0) + E[7],
		(E[8] * x0) + (E[9] * y0) + (E[10] * z0) + E[11]
	];

	//if (V[2] < 0)
	//	return null;

	const P = [
		(I[0] * V[0]) + (I[1] * V[1]) + (I[2] * V[2]),
		(I[3] * V[0]) + (I[4] * V[1]) + (I[5] * V[2]),
		(I[6] * V[0]) + (I[7] * V[1]) + (I[8] * V[2]),
	];

	return [ P[0] / P[2], P[1] / P[2] ];
};

class Spine {

	constructor (R, r, n) {

		this.R = R;
		this.r = r;
		this.n = n;
		this.o = [0, 0, 0];
	}

	moveTo (v) {
		this.o = [...v];
	}

	fx (t) {
		return (this.R + this.r * Math.cos(this.n * t)) * Math.cos(t);
	}

	fy (t) {
		return (this.R + this.r * Math.cos(this.n * t)) * Math.sin(t);
	}

	fz (t) {
		return this.r * Math.sin(this.n * t);
	}
}


let cnv, ctx;

const canvasRef = useTemplateRef("canvas-ref");

const spine = new Spine(100, 20, 13);

const generate = () => {

	const points = [];

	const target = vec(0, 0, 0);
	const camera = vec(-90, -90, 130);
	const f = 100;

	const dt = 0.1 / spine.n;

	for (let t = 0; t < pi2; t += dt) {

		points.push( proj( spine.fx(t), spine.fy(t), spine.fz(t), target, camera, f ) );
	}

	return points;
};

const render = (points, i0) => {

	ctx.clearRect(0, 0, cnv.width, cnv.height);

	ctx.fillStyle = "#ffffff";

	const gap = 0.1;

	let i = Math.floor( points.length * (gap + i0) );
	const ie = points.length + Math.floor( points.length * i0 );

	for (i; i < ie; i++) {

		const mi = i % points.length;

		const coef = -2 * Math.abs( (mi / points.length) - 0.5 ) + 1;
		const r = coef * 4 + 4;

		ctx.beginPath();

		ctx.arc(points[mi][0], points[mi][1], r, 0, pi2);
		ctx.fill();
	}
};

onMounted(() => {

	const points = generate();

	cnv = canvasRef.value;

	cnv.width = window.innerWidth;
	cnv.height = window.innerHeight;

	ctx = canvasRef.value.getContext("2d");

	const fps = 60;
	const dms = 1000 / fps;
	let _ms = new Date();

	let t = 0;
	const dt = 0.05 / fps;

	const animation = () => {

		const ms = new Date();

		if ( ms - _ms > dms ) {

			render(points, t);

			t += dt;
			t %= 1;

			_ms = ms;
		}

		requestAnimationFrame(animation);
	};

	requestAnimationFrame(animation);
});

</script>

<template>
	<canvas ref="canvas-ref"></canvas>
</template>

<style scoped>
canvas {
	background-color: black;
	display: block;
}
</style>
