## Dogs and cats classifiers with gradio

Dogs and cats classifier implementation using Hugging Face Spaces and Gradio based on the [Fastai](https://course.fast.ai/Lessons/lesson2.html) course.

<input id="photos" type="file" multiple="">
<script>
  async function loaded(reader) {
    const response = await fetch('https://chaozn-fastai-dogs-vs-cats.hf.space/api/predict', {
      method: "POST", body: JSON.stringify({ "data": [reader.result] }),
      headers: { "Content-Type": "application/json" }
    });
    const json = await response.json();
    const label = json['data'][0]['confidences'][0]['label'];
    const div = document.createElement('div');
    div.innerHTML = `<br/><img src="${reader.result}" width="300"> <p>${label}</p>`
    document.body.append(div);
  }
  function read(file) {
    const reader = new FileReader();
    reader.addEventListener('load', () => loaded(reader))
    reader.readAsDataURL(file);
  }
  photos.addEventListener('input', () => { [...photos.files].map(read) });
</script>
