<!DOCTYPE html>
<html lang="en">
<head>
  <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
</head>
<body style="background-color:black; color:white">
  <div id="plot" style="width:100%; height:100vh;"></div>
  <script>
    const x = [...Array(1000).keys()];
    const y = x.map(k => Math.sin(k * 0.01));

    const trace = {
      x: x,
      y: y,
      mode: 'markers',
      marker: {color: 'cyan'},
      type: 'scatter'
    };

    const layout = {
      paper_bgcolor: 'black',
      plot_bgcolor: 'black',
      xaxis: {title: 'k', color: 'white'},
      yaxis: {title: 'Value', color: 'white'},
      dragmode: 'zoom'
    };

    Plotly.newPlot('plot', [trace], layout);
  </script>
</body>
</html>
