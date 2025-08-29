<script setup>
import { onMounted, ref, useTemplateRef, watch } from "vue";

const canvasRef = useTemplateRef("canvas-ref");

const vec = (x, y, z) => ({ x, y, z });

const cross = (v1, v2) => ({
	x: (v1.y * v2.z) - (v1.z * v2.y),
	y: (v1.z * v2.x) - (v1.x * v2.z),
	z: (v1.x * v2.y) - (v1.y * v2.x)
});

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

const mxm = (m1, m2) => {

	return [
		(m1[0] * m2[0]) + (m1[1] * m2[3]) + (m1[2] * m2[6]), (m1[0] * m2[0]) + (m1[1] * m2[3]) + (m1[2] * m2[6]), (m1[0] * m2[0]) + (m1[1] * m2[3]) + (m1[2] * m2[6]),
		(m1[3] * m2[1]) + (m1[4] * m2[4]) + (m1[5] * m2[7]), (m1[3] * m2[1]) + (m1[4] * m2[4]) + (m1[5] * m2[7]), (m1[3] * m2[1]) + (m1[4] * m2[4]) + (m1[5] * m2[7]),
		(m1[6] * m2[2]) + (m1[7] * m2[5]) + (m1[8] * m2[8]), (m1[6] * m2[2]) + (m1[7] * m2[5]) + (m1[8] * m2[8]), (m1[6] * m2[2]) + (m1[7] * m2[5]) + (m1[8] * m2[8])
	];
};

const center = v => {

	if (v.length) {

		return [v[0] + O.x, v[1] + O.y];
	} else {
		
		return [v.x + O.x, v.y + O.y];
	}
};

const posX = ref(0);
const posY = ref(0);
const posZ = ref(0);

const O = { x: window.innerWidth / 2, y: window.innerHeight / 2 };
const trg = vec(0, 0, 0);
const cam = vec(0, 100, 100);
const up = vec(0, 1, 0);
const asp = window.innerWidth / window.innerHeight;
const fov = 1;

const proj = (x0, y0, z0) => {

	const L = norm(diff(trg, cam));
	const s = norm(cross(L, up));
	const u = cross(s, L);

	const x = (x0 * s.x) + (y0 * s.y) + (z0 * s.z);
	const y = (x0 * u.x) + (y0 * u.y) + (z0 * u.z);

	return [x * fov, y * fov];
};

const R = 100;
const n = 14;
const r = 20;
const pi2 = Math.PI * 2;
const O3d = vec(0, 0, 0);

const fx = t => (R + r * Math.cos(n * t)) * Math.cos(t);
const fy = t => (R + r * Math.cos(n * t)) * Math.sin(t);
const fz = t => r * Math.sin(n * t);

onMounted(() => {

	const cnv = canvasRef.value;

	cnv.width = window.innerWidth;
	cnv.height = window.innerHeight;

	const ctx = canvasRef.value.getContext("2d");

	ctx.strokeStyle = "#ffffff";
	ctx.fillStyle = "#ffffff";
	ctx.lineCap = "round";

	const peaks = 5;
	const w0 = 10 * Math.PI / 180;
	const dw = Math.PI * 2 / peaks;
	const ppRx = 100;
	const ppRy = 150;
	const cpRx = ppRx * 0.05;
	const cpRy = ppRy * 0.05;
	const dz = 0;

	let w = w0;

	const spark = [ppRx * Math.cos(w), ppRy * Math.sin(w), dz];

	for (w; w < pi2; w += dw) {

		spark.push(
			cpRx * Math.cos(w), cpRy * Math.sin(w), dz,
			cpRx * Math.cos(w + dw), cpRy * Math.sin(w + dw), dz,
			ppRx * Math.cos(w + dw), ppRy * Math.sin(w + dw), dz
		);
	}

	let t0 = 0;

	const anim = () => {

		//const gap = Math.PI / 5 + t0;
		const gap = 0;
		let t = -t0;
		const dt = 0.1 / n;
	
		let _xy = proj( O3d.x + fx(t), O3d.y + fy(t), O3d.z + fz(t) );

		ctx.clearRect(0, 0, cnv.width, cnv.height);

		for (t; t < pi2 + dt - gap; t += dt) {

			//ctx.lineWidth = 8 * Math.abs( (2 * t / pi2) - 1 ) + 8;
			ctx.lineWidth = 1;

			ctx.beginPath();
			ctx.moveTo(O.x + _xy[0], O.y + _xy[1]);

			_xy = proj( O3d.x + fx(t), O3d.y + fy(t), O3d.z + fz(t) );
			ctx.lineTo(O.x + _xy[0], O.y + _xy[1]);

			ctx.stroke();
		}

		//ctx.beginPath();
		//ctx.moveTo(...center( proj( spark[0], spark[1], spark[2] ) ) );

		//for (let i = 3; i < spark.length - 8; i += 9) {

		//	ctx.bezierCurveTo(
		//		...center( proj( spark[i], spark[i + 1], spark[i + 2] ) ),
		//		...center( proj( spark[i + 3], spark[i + 4], spark[i + 5] ) ),
		//		...center( proj( spark[i + 6], spark[i + 7], spark[i + 8] ) )
		//	);
		//}

		//ctx.lineWidth = 1;
		//ctx.fill();
		//ctx.stroke();

		//t0 = (t0 + 0.01) % pi2;

		//requestAnimationFrame(anim);
	};

	//requestAnimationFrame(anim);
	anim();

});
</script>

<template>
	<canvas ref="canvas-ref"></canvas>
	<div>
		<p>x: {{ posX }}</p>
		<input :value="posX" @change="e => posX = +e.target.value" type="range" min="-400" max="400" step="1">
		<p>y: {{ posY }}</p>
		<input :value="posY" @change="e => posY = +e.target.value" type="range" min="-400" max="400" step="1">
		<p>z: {{ posZ }}</p>
		<input :value="posZ" @change="e => posZ = +e.target.value" type="range" min="-400" max="400" step="1">
	</div>
</template>

<style scoped>
canvas {
	background-color: black;
	display: block;
}
div {
	position: fixed;
	top: 0;
	right: 0;
	color: aliceblue;
}
p {
	margin: 0;
}
input {
	display: block;
}
</style>
