<script setup>
import fitCurve from "fit-curve";
import { onMounted, onUpdated, ref, useTemplateRef, watch } from "vue";
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

const render = () => {

	const target = vec(0, 0, 0);
	const camera = vec(-90, -90, 130);
	const f = 100;

	ctx.clearRect(0, 0, cnv.width, cnv.height);

	const dt = 0.1 / spine.n;

	const points = [];

	for (let t = 0; t < pi2; t += dt) {

		points.push( proj( spine.fx(t), spine.fy(t), spine.fz(t), target, camera, f ) );
	}

	const plen = points.length;

	ctx.strokeStyle = "#ffffff";
	ctx.lineCap = "round";

	for (let i = 0; i < plen; i++) {

		const perc = -2 * Math.abs( (i / plen) - 0.5 ) + 1;
		ctx.lineWidth = 4 + (perc * 12);
		//ctx.lineWidth = 1;

		ctx.beginPath();

		if (i === 0) {
			ctx.moveTo(points[plen - 1][0], points[plen - 1][1]);
		} else {
			ctx.moveTo(points[i - 1][0], points[i - 1][1]);
		}
		ctx.lineTo(points[i][0], points[i][1]);
		ctx.stroke();
	}

	const curves = fitCurve(points, 2);

	ctx.lineWidth = 1;
	ctx.strokeStyle = "red";
	ctx.beginPath();
	ctx.moveTo(curves[0][0], curves[0][1]);

	for (let i = 0; i < curves.length; i++) {

		ctx.bezierCurveTo(...curves[i][1], ...curves[i][2], ...curves[i][3]);
	}

	ctx.stroke();

	ctx.beginPath();

	ctx.fillStyle = "red";

	for (let i = 0; i < curves.length; i++) {

		ctx.moveTo(...curves[i][0]);
		ctx.arc(...curves[i][0], 2, 0, pi2);
	}

	ctx.fill();


	const Ox = window.innerWidth / 2, Oy = window.innerHeight / 2;

	console.log(
		`<?xml version="1.0" standalone="no"?>\n<svg width="5cm" height="4cm" version="1.1" xmlns="http://www.w3.org/2000/svg">\n\t<path d="M ${
			(curves[0][0][0] - Ox).toFixed(4)
		} ${
			(curves[0][0][1] - Oy).toFixed(4)
		}${
			curves.map(([, b, c, d ]) => {

				return ` C ${
						(b[0] - Ox).toFixed(4)
					} ${
						(b[1] - Oy).toFixed(4)
					}, ${
						(c[0] - Ox).toFixed(4)
					} ${
						(c[1] - Oy).toFixed(4)
					}, ${
						(d[0] - Ox).toFixed(4)
					} ${
						(d[1] - Oy).toFixed(4)
					}`;
			})
		}"/>\n</svg>`
	);
};

onMounted(() => {

	cnv = canvasRef.value;

	cnv.width = window.innerWidth;
	cnv.height = window.innerHeight;

	ctx = canvasRef.value.getContext("2d");

	render();
});

//onUpdated(() => {

//	render();
//});

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
