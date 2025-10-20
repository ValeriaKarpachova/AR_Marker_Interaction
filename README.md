<section style="font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial; line-height:1.5; color:#111;">
  <h1 style="margin-bottom:0.1em;">AR Marker Interaction</h1>
  <p style="margin-top:0.2em; color:#444;">
  Програма на Python, що використовує яка використовує <b>OpenCV</b> і <b>ArUco</b> для створення ефектів доповненої реальності.
  Кожен маркер виконує окрему дію: відтворює відео, відкриває Spotify або завершує роботу програми.
</p>
<p>
  Цей проєкт був розроблений та адаптований <a href="https://github.com/codegiovanni/Augmentation_Aruco_marker.git" target="_blank">Augmentation_Aruco_marker</a> під демонстрацію інтерактивної AR-взаємодії з маркерами.
</p>
<div align="center">
  <img src="result.gif" alt="AR Marker Interaction"/>
</div>
  <ul style="color:#333">
    <li><b>Маркeр 7</b> — показує анімоване відео (Donut.gif) поверх маркера;</li>
    <li><b>Маркeр 13</b> — відкриває трек у Spotify;</li>
    <li><b>Маркeр 42</b> — завершує роботу програми.</li>
  </ul>
  <div align="center">
  <div style="display: flex; justify-content: center; gap: 10px;">
  <img src="img1.png" alt="Marker 1" style="width: 30%;">
  <img src="img2.png" alt="Marker 2" style="width: 30%;">
  <img src="img3.png" alt="Marker 3" style="width: 30%;">
</div>

  </div>

  <hr style="border:none;border-top:1px solid #eee;margin:16px 0;">

  <h2 style="font-size:1.15rem;margin-bottom:6px;">Функціонал</h2>
  <ul style="margin-top:0;color:#333">
    <li>Реальне розпізнавання ArUco-маркера через камеру.</li>
    <li>Накладання відео на площину маркера (AR-ефект).</li>
    <li>Автоматичне відкриття посилання на Spotify.</li>
    <li>Завершення програми спеціальним маркером-тригером.</li>
  </ul>

  <hr style="border:none;border-top:1px solid #eee;margin:16px 0;">

  <h2 style="font-size:1.15rem;margin-bottom:6px;">Пояснення коду</h2>

  <section style="font-family: Consolas, monospace; background:#f9f9f9; border-radius:10px; padding:16px; border:1px solid #ddd;">

<b>Імпорт необхідних бібліотек</b>
  <pre style="background:#fff; padding:10px; border-radius:8px; border:1px solid #ccc;">
<code>import cv2
import cv2.aruco as aruco
import numpy as np
import webbrowser
import time
</code></pre>
  <hr>

  <b>Оголошення ID маркерів</b>
  <pre style="background:#fff; padding:10px; border-radius:8px; border:1px solid #ccc;">
<code>id_marker = 7
id_marker2 = 13
id_marker3 = 42
</code></pre>
  <hr>

  <b>Створення словника ArUco та параметрів детектора</b>
  <pre style="background:#fff; padding:10px; border-radius:8px; border:1px solid #ccc;">
<code>aruco_dict = aruco.getPredefinedDictionary(aruco.DICT_4X4_50)
parameters = aruco.DetectorParameters()
</code></pre>
  <hr>

  <b>Завантаження ресурсів (відео та посилання Spotify)</b>
  <pre style="background:#fff; padding:10px; border-radius:8px; border:1px solid #ccc;">
<code>video_augment = cv2.VideoCapture("Donut.gif")
spotify_track_url = "https://open.spotify.com/track/0pqnGHJpmpxLKifKRmU6WP"
</code></pre>
  <hr>

  <b>Ініціалізація камери та службових змінних</b>
  <pre style="background:#fff; padding:10px; border-radius:8px; border:1px solid #ccc;">
<code>cap = cv2.VideoCapture(0 + cv2.CAP_DSHOW)
detection = False
frame_count = 0
music_played = False
</code></pre>
  <hr>

  <b>Зчитування першого кадру з GIF-відео</b>
  <pre style="background:#fff; padding:10px; border-radius:8px; border:1px solid #ccc;">
<code>_, image_video = video_augment.read()
image_video = cv2.resize(image_video, (100, 100))
</code></pre>
  <hr>

  <b>Функція для накладання зображення на поверхню маркера</b>
  <pre style="background:#fff; padding:10px; border-radius:8px; border:1px solid #ccc;">
<code>def augmentation(bbox, img, img_augment):
    top_left = bbox[0][0][0], bbox[0][0][1]
    top_right = bbox[0][1][0], bbox[0][1][1]
    bottom_right = bbox[0][2][0], bbox[0][2][1]
    bottom_left = bbox[0][3][0], bbox[0][3][1]

    height, width, _ = img_augment.shape
    points_1 = np.array([top_left, top_right, bottom_right, bottom_left])
    points_2 = np.float32([[0, 0], [width, 0], [width, height], [0, height]])

    matrix, _ = cv2.findHomography(points_2, points_1)
    image_out = cv2.warpPerspective(img_augment, matrix, (img.shape[1], img.shape[0]))
    cv2.fillConvexPoly(img, points_1.astype(int), (0, 0, 0))
    return img + image_out
</code></pre>
  <hr>

  <b>Основний цикл програми (обробка кадрів)</b>
  <pre style="background:#fff; padding:10px; border-radius:8px; border:1px solid #ccc;">
<code>while True:
    _, frame = cap.read()
    if not detection:
        video_augment.set(cv2.CAP_PROP_POS_FRAMES, 0)
        frame_count = 0
    else:
        if frame_count == video_augment.get(cv2.CAP_PROP_FRAME_COUNT):
            video_augment.set(cv2.CAP_PROP_POS_FRAMES, 0)
            frame_count = 0
        _, image_video = video_augment.read()
        image_video = cv2.resize(image_video, (100, 100))

    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    corners, ids, rejected = aruco.detectMarkers(image=gray, dictionary=aruco_dict, parameters=parameters)
</code></pre>
  <hr>

  <b>Перевірка знайдених маркерів і виконання дій</b>
  <pre style="background:#fff; padding:10px; border-radius:8px; border:1px solid #ccc;">
<code>    if ids is not None:
        for i, marker_id in enumerate(ids.flatten()):
            if marker_id == id_marker:
                detection = True
                frame = augmentation(np.array(corners)[i], frame, image_video)

            elif marker_id == id_marker2 and not music_played:
                webbrowser.open(spotify_track_url)
                music_played = True
                time.sleep(2)

            elif marker_id == id_marker3:
                print("Маркeр завершення виявлено. Програму зупинено.")
                cap.release()
                cv2.destroyAllWindows()
                exit()
</code></pre>
  <hr>

  <b>Відображення кадру та завершення програми</b>
  <pre style="background:#fff; padding:10px; border-radius:8px; border:1px solid #ccc;">
<code>    cv2.imshow('input', frame)
    if cv2.waitKey(1) & 0xFF == 27:
        break
    frame_count += 1

cap.release()
cv2.destroyAllWindows()
</code></pre>

</section>
