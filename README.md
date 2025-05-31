# geo tagging photo 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Photo GeoTagger - Modern AI Style</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');
  body {
    margin: 0; padding: 0;
    font-family: 'Poppins', sans-serif;
    background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
    color: #e0e6f0;
    display: flex;
    flex-direction: column;
    align-items: center;
    min-height: 100vh;
    padding: 3rem 1.5rem;
    user-select: none;
  }
  h1 {
    margin-bottom: 0.3rem;
    font-weight: 700;
    font-size: 3rem;
    text-transform: uppercase;
    letter-spacing: 0.12em;
    text-shadow: 0 3px 8px rgba(255,255,255,0.15);
  }
  p.description {
    margin-top: 0;
    margin-bottom: 3rem;
    font-size: 1.1rem;
    color: #a8b0c3;
    max-width: 440px;
    text-align: center;
    line-height: 1.5;
  }
  #photoInput {
    background: linear-gradient(90deg, #7F00FF, #E100FF);
    color: white;
    padding: 1rem 2rem;
    font-size: 1.1rem;
    border-radius: 30px;
    cursor: pointer;
    border: none;
    box-shadow: 0 10px 25px -5px rgba(225,0,255,0.6);
    transition: all 0.3s ease;
    width: 280px;
    display: block;
    margin: 0 auto 2rem auto;
    font-weight: 600;
    text-align: center;
  }
  #photoInput:hover, #photoInput:focus {
    background: linear-gradient(90deg, #c56aff, #ff5aff);
    outline: none;
    box-shadow: 0 14px 30px -7px rgba(255,90,255,0.8);
  }
  button#geotagBtn {
    margin-top: 1rem;
    background: linear-gradient(90deg, #00F260, #0575E6);
    border: none;
    padding: 1rem 3rem;
    border-radius: 40px;
    font-size: 1.2rem;
    font-weight: 700;
    color: #fff;
    cursor: pointer;
    box-shadow: 0 10px 30px -5px rgba(5,117,230,0.8);
    transition: background 0.3s ease, transform 0.2s ease;
    position: relative;
    overflow: hidden;
    outline-offset: 4px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    user-select: none;
  }
  button#geotagBtn:disabled {
    background: #4a4a4a;
    box-shadow: none;
    cursor: not-allowed;
  }
  button#geotagBtn:hover:not(:disabled) {
    background: linear-gradient(90deg, #05e676, #46a0ff);
    transform: scale(1.05);
  }
  #status {
    margin-top: 1.2rem;
    font-weight: 600;
    font-size: 1rem;
    min-height: 1.3rem;
    max-width: 320px;
    text-align: center;
    user-select: text;
  }
  #previewContainer {
    position: relative;
    margin-top: 2rem;
    max-width: 360px;
    max-height: 360px;
    border-radius: 20px;
    box-shadow: 0 10px 28px rgba(0,0,0,0.85);
    user-select: none;
  }
  #preview {
    display: block;
    width: 100%;
    height: auto;
    border-radius: 20px;
    object-fit: contain;
  }
  #locationOverlay {
    position: absolute;
    bottom: 14px;
    right: 14px;
    background: rgba(0,0,0,0.55);
    backdrop-filter: blur(8px);
    -webkit-backdrop-filter: blur(8px);
    padding: 8px 14px;
    border-radius: 14px;
    font-size: 1rem;
    font-weight: 600;
    color: #7afcff;
    text-shadow: 0 0 6px #7afcff90;
    user-select: text;
    max-width: 90%;
    line-break: anywhere;
    display: none;
  }
  a#downloadLink {
    margin-top: 2rem;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    background: linear-gradient(135deg, #ff416c, #ff4b2b);
    color: white;
    font-weight: 700;
    text-decoration: none;
    padding: 1rem 2.5rem;
    border-radius: 50px;
    box-shadow: 0 14px 45px -10px rgba(255,75,43,0.8);
    font-size: 1.1rem;
    user-select: none;
    transition: background 0.3s ease, transform 0.2s ease;
  }
  a#downloadLink:hover {
    background: linear-gradient(135deg, #ff5a82, #ff704c);
    transform: scale(1.07);
  }
  a#downloadLink:active {
    transform: scale(0.97);
  }
  /* Loading spinner inside the GeoTag button */
  .spinner {
    border: 3.5px solid rgba(255,255,255,0.25);
    border-top: 3.5px solid #fff;
    border-radius: 50%;
    width: 22px;
    height: 22px;
    animation: spin 1s linear infinite;
    margin-left: 10px;
  }
  @keyframes spin {
    to { transform: rotate(360deg); }
  }
</style>
</head>
<body>
  <h1>Photo GeoTagger</h1>
  <p class="description">Upload a photo, allow location access, and embed your current geolocation coordinates into the photo's metadata. Then download your geo-tagged photo.</p>
  <input type="file" id="photoInput" accept="image/jpeg,image/jpg" />
  <button id="geotagBtn" disabled>Geo Tag Photo</button>
  <div id="status"></div>

  <div id="previewContainer" aria-live="polite" aria-atomic="true">
    <img id="preview" alt="Photo preview" />
    <div id="locationOverlay" aria-label="Geo-tagged location coordinates"></div>
  </div>

  <a id="downloadLink" href="#" download="" style="display:none;">&#128229; Download GeoTagged Photo</a>

  <!-- Include piexifjs from CDN -->
  <script src="https://cdn.jsdelivr.net/npm/piexifjs"></script>
  <script>
    const photoInput = document.getElementById('photoInput');
    const geotagBtn = document.getElementById('geotagBtn');
    const status = document.getElementById('status');
    const preview = document.getElementById('preview');
    const downloadLink = document.getElementById('downloadLink');
    const locationOverlay = document.getElementById('locationOverlay');

    let originalDataURL = null;
    let originalFileName = '';

    function resetUI() {
      status.textContent = "";
      preview.style.display = "none";
      locationOverlay.style.display = "none";
      locationOverlay.textContent = "";
      downloadLink.style.display = "none";
      downloadLink.href = "#";
      downloadLink.download = "";
      geotagBtn.disabled = true;
      geotagBtn.innerHTML = "Geo Tag Photo";
      geotagBtn.style.pointerEvents = "auto";
    }

    photoInput.addEventListener('change', () => {
      const file = photoInput.files[0];
      resetUI();
      if(file && (file.type === "image/jpeg" || file.type === "image/jpg")) {
        originalFileName = file.name;
        const reader = new FileReader();
        reader.onload = function(e) {
          originalDataURL = e.target.result;
          preview.src = originalDataURL;
          preview.style.display = "block";
          geotagBtn.disabled = false;
          status.textContent = "Photo loaded. Ready to geo tag.";
        };
        reader.readAsDataURL(file);
      } else {
        status.textContent = "Please select a JPEG photo.";
        geotagBtn.disabled = true;
      }
    });

    geotagBtn.addEventListener('click', () => {
      if(!originalDataURL) return;
      status.textContent = "Getting your location...";
      geotagBtn.disabled = true;
      geotagBtn.innerHTML = `Geo Tagging... <span class="spinner"></span>`;
      geotagBtn.style.pointerEvents = "none";
      locationOverlay.style.display = "none";
      locationOverlay.textContent = "";

      if (!navigator.geolocation) {
        status.textContent = "Geolocation is not supported by your browser.";
        geotagBtn.disabled = false;
        geotagBtn.innerHTML = "Geo Tag Photo";
        geotagBtn.style.pointerEvents = "auto";
        return;
      }

      navigator.geolocation.getCurrentPosition(success, error, { enableHighAccuracy: true, timeout: 10000, maximumAge: 0 });

      function success(position) {
        const lat = position.coords.latitude;
        const lon = position.coords.longitude;
        status.textContent = `Location acquired: (${lat.toFixed(5)}, ${lon.toFixed(5)}). Geo tagging photo...`;

        try {
          const zeroth = {};
          const exif = {};
          const gps = {};

          // convert decimal degrees to EXIF rational numbers (degrees, minutes, seconds)
          function toRational(number) {
            const denominator = 1000000;
            return [Math.round(number * denominator), denominator];
          }

          function decimalToDMS(degFloat) {
            const deg = Math.floor(Math.abs(degFloat));
            const minFloat = (Math.abs(degFloat) - deg) * 60;
            const min = Math.floor(minFloat);
            const sec = (minFloat - min) * 60;
            return [
              toRational(deg),
              toRational(min),
              toRational(sec)
            ];
          }

          gps[piexif.GPSIFD.GPSLatitudeRef] = lat >= 0 ? "N" : "S";
          gps[piexif.GPSIFD.GPSLatitude] = decimalToDMS(lat);
          gps[piexif.GPSIFD.GPSLongitudeRef] = lon >= 0 ? "E" : "W";
          gps[piexif.GPSIFD.GPSLongitude] = decimalToDMS(lon);

          const exifObj = {"0th": zeroth, "Exif": exif, "GPS": gps};
          const exifBytes = piexif.dump(exifObj);
          const newDataUrl = piexif.insert(exifBytes, originalDataURL);

          preview.src = newDataUrl;

          downloadLink.href = newDataUrl;
          const splittedName = originalFileName.split('.');
          const ext = splittedName.pop();
          const baseName = splittedName.join('.');
          downloadLink.download = baseName + "_geotagged." + ext;
          downloadLink.style.display = "inline-flex";

          // Show coordinates overlay on the photo
          locationOverlay.textContent = `Lat: ${lat.toFixed(6)}°, Lon: ${lon.toFixed(6)}°`;
          locationOverlay.style.display = "block";

          status.textContent = "Photo geo tagged successfully! You can download it now.";
        } catch (err) {
          status.textContent = "Error while geo tagging: " + err.message;
          locationOverlay.style.display = "none";
          locationOverlay.textContent = "";
          downloadLink.style.display = "none";
        }
        geotagBtn.disabled = false;
        geotagBtn.innerHTML = "Geo Tag Photo";
        geotagBtn.style.pointerEvents = "auto";
      }

      function error(err) {
        switch(err.code) {
          case err.PERMISSION_DENIED:
            status.textContent = "Permission denied. Please allow location access.";
            break;
          case err.POSITION_UNAVAILABLE:
            status.textContent = "Position unavailable. Please try again.";
            break;
          case err.TIMEOUT:
            status.textContent = "Location request timed out. Please try again.";
            break;
          default:
            status.textContent = "Unknown error occurred.";
        }
        geotagBtn.disabled = false;
        geotagBtn.innerHTML = "Geo Tag Photo";
        geotagBtn.style.pointerEvents = "auto";
        locationOverlay.style.display = "none";
        locationOverlay.textContent = "";
        downloadLink.style.display = "none";
      }
    });
  </script>
</body>
</html>


