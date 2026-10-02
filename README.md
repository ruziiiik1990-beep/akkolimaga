<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AK-47 Rat Rod Cloud</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { overflow: hidden; background-color: #050505; width: 100vw; height: 100vh; }
        canvas { display: block; width: 100%; height: 100%; }
    </style>
    <!-- Подключаем Three.js -->
    <script src="https://cloudflare.com"></script>
</head>
<body>

<canvas class="webgl"></canvas>

<script>
    const canvas = document.querySelector('canvas.webgl');
    const scene = new THREE.Scene();

    const parameters = {
        size: 0.04, // Размер твоих точек
    };

    let geometry = null;
    let material = null;
    let points = null;

    const generateWeaponParticles = () => {
        // ТВОЙ КОД ОЧИСТКИ СЦЕНЫ
        if(points !== null){
            geometry.dispose();
            material.dispose();
            scene.remove(points);
        }

        const image = new Image();
        
        // ВАЖНО: Относительный путь к файлу в твоем репозитории полностью убирает ошибку CORS!
        image.src = "ak-47_kolymaga.png";

        image.onload = () => {
            const imgCanvas = document.createElement('canvas');
            const ctx = imgCanvas.getContext('2d');
            
            // Настройка плотности облака точек (150 точек по ширине — оптимально для производительности)
            const width = 150;
            const height = Math.round((image.height / image.width) * width);
            imgCanvas.width = width;
            imgCanvas.height = height;
            
            ctx.drawImage(image, 0, 0, width, height);
            const imgData = ctx.getImageData(0, 0, width, height).data;

            const positions = [];
            const colors = [];

            for (let y = 0; y < height; y++) {
                for (let x = 0; x < width; x++) {
                    const index = (y * width + x) * 4;
                    const r = imgData[index] / 255;
                    const g = imgData[index + 1] / 255;
                    const b = imgData[index + 2] / 255;
                    const alpha = imgData[index + 3] / 255;

                    // Отсекаем прозрачный фон картинки
                    if (alpha > 0.2) {
                        // Центрируем автомат по осям X и Y
                        const posX = (x - width / 2) * 0.045;
                        const posY = -(y - height / 2) * 0.045;
                        // Небольшой случайный разброс по оси Z, чтобы модель выглядела объемной при вращении
                        const posZ = (Math.random() - 0.5) * 0.15; 

                        positions.push(posX, posY, posZ);
                        colors.push(r, g, b);
                    }
                }
            }

            geometry = new THREE.BufferGeometry();
            geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));
            geometry.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3));

            // ТВОЙ МАТЕРИАЛ POINTSMATERIAL С АДДИТИВНЫМ СВЕЧЕНИЕМ
            material = new THREE.PointsMaterial({
                size: parameters.size,
                sizeAttenuation: true,
                depthWrite: false,
                blending: THREE.AdditiveBlending,
                vertexColors: true
            });

            points = new THREE.Points(geometry, material);
            scene.add(points);
        };

        image.onerror = () => {
            console.error("Не удалось найти файл ak-47_kolymaga.png в репозитории!");
        };
    };

    generateWeaponParticles();

    // Размеры контейнера фрейма
    const sizes = { width: canvas.clientWidth, height: canvas.clientHeight };
    
    const camera = new THREE.PerspectiveCamera(65, sizes.width / sizes.height, 0.1, 100);
    camera.position.z = 4.5; // Дистанция камеры до автомата
    scene.add(camera);

    const renderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: true });
    renderer.setSize(sizes.width, sizes.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

    // Адаптивное обновление размеров при изменении окна фрейма
    window.addEventListener('resize', () => {
        sizes.width = canvas.clientWidth;
        sizes.height = canvas.clientHeight;
        
        camera.aspect = sizes.width / sizes.height;
        camera.updateProjectionMatrix();
        
        renderer.setSize(sizes.width, sizes.height);
    });

    // Анимационный цикл (Бесконечное 360 вращение)
    const clock = new THREE.Clock();

    const tick = () => {
        const elapsedTime = clock.getElapsedTime();

        if(points) {
            // Вращаем автомат из точек вокруг вертикальной оси Y
            points.rotation.y = elapsedTime * 0.4; 
        }

        renderer.render(scene, camera);
        window.requestAnimationFrame(tick);
    };

    tick();
</script>

</body>
</html>
