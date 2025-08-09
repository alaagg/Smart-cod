<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>تروس تفاعلية باللمس</title>
<style>
    body { margin: 0; overflow: hidden; background: #111; }
    canvas { display: block; }
</style>
</head>
<body>

<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>

<script>
let scene, camera, renderer;
let gear1, gear2;
let isDragging = false, activeGear = null;
let prevTouchX = 0;

init();
animate();

function createGear(radius, teeth, color) {
    // جسم الترس: أسطوانة بسيطة (تمثيل بصري)
    let geom = new THREE.CylinderGeometry(radius, radius, 4, teeth * 4);
    let mat = new THREE.MeshStandardMaterial({ color: color, metalness: 0.7, roughness: 0.3 });
    let mesh = new THREE.Mesh(geom, mat);
    mesh.rotation.x = Math.PI / 2;
    return mesh;
}

function init() {
    scene = new THREE.Scene();
    camera = new THREE.PerspectiveCamera(45, window.innerWidth/window.innerHeight, 0.1, 1000);
    camera.position.set(0, 40, 60);
    camera.lookAt(0,0,0);

    renderer = new THREE.WebGLRenderer({antialias:true});
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // إضاءة
    let light = new THREE.PointLight(0xffffff, 1);
    light.position.set(50, 50, 50);
    scene.add(light);
    scene.add(new THREE.AmbientLight(0x555555));

    // إنشاء ترسين
    gear1 = createGear(10, 12, 0xff4444);
    gear2 = createGear(10, 12, 0x44aaff);

    // مواقع التروس (متشابكة)
    gear1.position.x = -11;
    gear2.position.x = 11;

    scene.add(gear1);
    scene.add(gear2);

    // أحداث اللمس
    renderer.domElement.addEventListener('touchstart', onTouchStart);
    renderer.domElement.addEventListener('touchmove', onTouchMove);
    renderer.domElement.addEventListener('touchend', onTouchEnd);
}

function onTouchStart(event) {
    if (event.touches.length === 1) {
        let touch = event.touches[0];
        prevTouchX = touch.clientX;

        // اختيار الترس الأقرب للمس
        let mouse = new THREE.Vector2(
            (touch.clientX / window.innerWidth) * 2 - 1,
            -(touch.clientY / window.innerHeight) * 2 + 1
        );
        let raycaster = new THREE.Raycaster();
        raycaster.setFromCamera(mouse, camera);
        let intersects = raycaster.intersectObjects([gear1, gear2]);
        if (intersects.length > 0) {
            activeGear = intersects[0].object;
            isDragging = true;
        }
    }
}

function onTouchMove(event) {
    if (isDragging && activeGear && event.touches.length === 1) {
        let touch = event.touches[0];
        let deltaX = touch.clientX - prevTouchX;
        prevTouchX = touch.clientX;

        // دوران الترس باللمس
        activeGear.rotation.z += deltaX * 0.02;

        // دوران الترس الآخر بعكس الاتجاه
        if (activeGear === gear1) {
            gear2.rotation.z -= deltaX * 0.02;
        } else {
            gear1.rotation.z -= deltaX * 0.02;
        }
    }
}

function onTouchEnd(event) {
    isDragging = false;
    activeGear = null;
}

function animate() {
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
}
</script>

</body>
</html>
