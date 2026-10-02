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
    <!-- Подключаем стабильную версию Three.js -->
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

    // Сразу инициализируем базовые размеры сцены
    const sizes = { width: window.innerWidth, height: window.innerHeight };
    
    const camera = new THREE.PerspectiveCamera(60, sizes.width / sizes.height, 0.1, 100);
    camera.position.z = 5; 
    scene.add(camera);

    const renderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: true });
    renderer.setSize(sizes.width, sizes.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

    // Функция генерации автомата из пикселей
    const generateWeaponParticles = (loadedImage) => {
        // ТВОЙ КОД ОЧИСТКИ СЦЕНЫ
        if(points !== null){
            geometry.dispose();
            material.dispose();
            scene.remove(points);
        }

        const imgCanvas = document.createElement('canvas');
        const ctx = imgCanvas.getContext('2d');
        
        // Устанавливаем плотность точек по ширине
        const width = 160;
        const height = Math.round((loadedImage.height / loadedImage.width) * width);
        imgCanvas.width = width;
        imgCanvas.height = height;
        
        ctx.drawImage(loadedImage, 0, 0, width, height);
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

                // Если пиксель не прозрачный — создаем 3D точку
                if (alpha > 0.2) {
                    // Центрируем модель автомата
                    const posX = (x - width / 2) * 0.045;
                    const posY = -(y - height / 2) * 0.045;
                    // Создаем объем с помощью случайного разброса частиц в глубину Z
                    const posZ = (Math.random() - 0.5) * 0.15; 

                    positions.push(posX, posY, posZ);
                    colors.push(r, g, b);
                }
            }
        }

        geometry = new THREE.BufferGeometry();
        geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));
        geometry.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3));

        // ТВОЙ МАТЕРИАЛ POINTSMATERIAL
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

    // СНАЧАЛА ГАРАНТИРОВАННО ЗАГРУЖАЕМ КАРТИНКУ
    const image = new Image();
    image.crossOrigin = "anonymous";
    // Файл обязательно должен лежать в той же папке на GitHub под этим именем!
    image.src = "ak-47_kolymaga.png"; 

    image.onload = () => {
        // Как только картинка полностью загрузилась — запускаем сборку точек
        generateWeaponParticles(image);
    };

    image.onerror = () => {
        console.error("Ошибка: Проверь, лежит ли файл ak-47_kolymaga.png в корне твоего репозитория на GitHub!");
    };

    // Адаптив при изменении размеров экрана фрейма
    window.addEventListener('resize', () => {
        sizes.width = window.innerWidth;
        sizes.height = window.innerHeight;
        
        camera.aspect = sizes.width / sizes.height;
        camera.updateProjectionMatrix();
        
        renderer.setSize(sizes.width, sizes.height);
    });

    // Анимационный цикл (Вращение на 360 градусов)
    const clock = new THREE.Clock();

    const tick = () => {
        const elapsedTime = clock.getElapsedTime();

        if(points) {
            // Крутим автомат вокруг вертикальной оси
            points.rotation.y = elapsedTime * 0.4; 
        }

        renderer.render(scene, camera);
        window.requestAnimationFrame(tick);
    };

    tick();
</script>

</body>
</html>
