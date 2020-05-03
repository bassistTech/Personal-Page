<html>
<head>
    <script language="javascript" type="text/javascript" src="flot/jquery.js"></script>
    <script language="javascript" type="text/javascript" src="flot/jquery.flot.js"></script>
</head>

<body>
    <h1>Speaker Modeling Program</h1>
	Revision date: March 9, 2016<br>
	<br>
	This is a proof-of-concept that I wrote to teach myself JavaScript. It's not intended for 
	serious design work. However, it uses the basic speaker modeling formulas that I've derived
	and tested in other programs.<br><br>
	
	You need to turn on JavaScript in your browser to see anything here. If you're paranoid, 
	you can look at the page source to make sure that everything is tame before turning on JavaScript. 
	This program uses Flot, an open source plotting library for JavaScript. You can find out more 
	at: <a href = "http://www.flotcharts.org/">www.flotcharts.org</a>. I'm using Flot in its entirety with no changes.<br><br>
	<h2>How did you get here?</h2>
	<OL>
		<LI>You entered a short URL (no parameters such as Fs, etc.) or clicked on a link. This means you're 
		looking at the defaults for	the program, my little Eminence 2512-ii box. You can enter your own 
		Thiele-Small and box parameters, and start modeling right away.
		<LI>You entered a long URL (lots of parameters) or clicked on such a link. This means you're looking
		at a driver model or box design that somebody has shared (or are using one of my library drivers).
		You can look at the model or change it and see what happens.
	</OL>
	A quick note about the P<sub>e</sub> parameter. <i>The program doesn't use it</i>. Instead, the program uses the Amplifier Power parameter for the excursion and port air speed graphs. This lets you figure out how your speaker will perform for your amplifier.
	<h2>Site Map</h2>
	<UL>
		<LI><a href="misc.html#dlibrary">Driver Library</a>
		<LI><a href="misc.html#faq">FAQ and troubleshooting</a>
		<LI><a href="#disclaimer">Copyright and warranty disclaimer</a>
		<LI><a href="misc.html#history">Revision history</a>
	</UL>
    <form name="example">
		<table>
			<script>
				// Read a page's GET URL variables and return them as an associative array.
				function getUrlVars()
				{
					var vars = [], hash;
					var hashes = window.location.href.slice(window.location.href.indexOf('?') + 1).split('&');
					for(var i = 0; i < hashes.length; i++)
					{
						hash = hashes[i].split('=');
						vars.push(hash[0]);
						vars[hash[0]] = hash[1];
					}
					return vars;
				}
				
				// Functions for creating form with multiple data entry boxes
				
				// Get a parameter from URL
				
				function getparm(s, defaultvalue) {
					t = decodeURI(location.search)
					if (t.length == 0) return defaultvalue // no parameters
					u = t.split(s + '=')
					if (u.length == 1) return defaultvalue
					return u[1].split('&')[0]
					//return location.search.split(se)[1] ? location.search.split(se)[1] : defaultvalue;
				}
				
				// Header line
				
				function formheader(textname) {
					document.write('<tr><td colspan="2"><h2>' + textname.toString() + '</h2></td></tr>')
				}
				
				// Data entry boxes
				
				function formparam(textname, javaname, initvalue, units) {
					document.write('<tr><td>' + textname.toString() + '</td><td><input name="' + 
						javaname.toString() + '" value="' + 
						initvalue.toString() + '"></td><td>' +
						units.toString() + '</td></tr>')
				}
				
				function formparama(textname, javaname, initvalue, units) {
					formparam(textname, javaname, getparm(javaname, initvalue), units)
				}
				
				// Create the form
				
				formheader('Driver Parameters')
				formparama('Driver name', 'Dname', 'Eminence 2512-ii', '')
				formparama('F<sub>s</sub>', 'Fs', '37', 'Hz')
				formparama('R<sub>e</sub>', 'Re', '5.04', 'Ohms')
				formparama('Q<sub>es</sub>', 'Qes', '0.44', '')
				formparama('Q<sub>ms</sub>', 'Qms', '3.13', '')
				formparama('V<sub>as</sub>', 'Vas', '147', 'Liters')
				formparama('S<sub>d</sub>', 'Sd', '519.5', 'cm<sup>2</sup>')
				formparama('X<sub>max</sub>', 'Xmax', '4.9', 'mm')
				formparama('L<sub>e</sub>', 'Le', '0.46', 'mH')
				formparama('Z<sub>nom</sub>', 'Znom', '8', 'Ohms')
				formparama('P<sub>e</sub>', 'Pe', '250', 'Watts')
				formheader('Box Parameters')
				formparama('Box design name', 'Bname', '32 liter 40 Hz box', '')
				formparama('Volume', 'V_liters', '32', 'Liters')
				formparama('Number of drivers', 'Ndrivers', '1', '')
				formparama('Port tuning', 'Fport', '40', 'Hz, set to 0 for sealed box')
				formparama('Port width', 'Aport', '5', 'cm')
				formparama('Port height', 'Bport', '8', 'cm')
				formparama('Number of ports', 'nPorts', '1', '')
				formparama('Port end correction', 'endCorrect', '0.732', '')
				formparama('Amplifier power', 'PinExc', '150', 'Watts')
			</script>
		</table>
		<p class="stackControls">
			<input type="button" value="Re-generate Graphs">
		</p>
	</form>
    
    <h2>Cone excursion in mm</h2>
    <div id="excursion" style="width:600px;height:300px;"></div>
	
    <h2>Sensitivity in dB SPL @ 1m</h2>
    <div id="sensitivity" style="width:600px;height:300px;"></div>
	
    <h2>Port air speed in mach</h2>
    <div id="mach" style="width:600px;height:300px;"></div>
	
    <h2>Reported Data</h2>
	<form name="after">
		<textarea cols="80" rows="20" name="output">Reported data</textarea><br><br>
		URL for this speaker. The following URL will reproduce your design. You can send this URL
		to a friend or save it in a file as way to "save" your design.<br><br>
		<textarea cols="80" rows="10" name="url">URL</textarea>
	</form>

<script type="text/javascript">
	
	// general purpose print to output form
	
	function dprint(s, v)
	{
		ts = document.forms['after'].output.value;
		ts += s + v + '\n';
		document.forms['after'].output.value = ts;
		return v
	}
	
	// get value from data entry form with echo to output form for debugging
	
	function dprinty(javaname) {
		v = document.forms['example'][javaname].value
		ts = document.forms['after'].url.value
		ts += javaname + '=' + encodeURI(v) + '&'
		document.forms['after'].url.value = ts
		return dprint(javaname + " = ", parseFloat(v))
	}
	
	function dprints(javaname) {
		v = document.forms['example'][javaname].value
		ts = document.forms['after'].url.value
		ts += javaname + '=' + encodeURI(v) + '&'
		document.forms['after'].url.value = ts
		return dprint(javaname + " = ", v)
	}

	// System parameters, OK to change

	PinSens = dprint("Power input for sensitivity graph = ", 1.0);
	Dist = dprint("Distance for SPL graph = ", 1.0);

	// Physical constants in SI units

	Patm = dprint("Atmospheric pressure = ", 101300);
	b = dprint("Adiabatic constant = ", 1.4);
	bpatm = dprint("bpatm = ", b*Patm);
	rho = dprint("Density of air = ", 1.18);
	c = dprint("Speed of sound = ", 345);
	splref = dprint("SPL reference = ", 2e-5);

	// Get stuff
	
	function getstuff()
	{
		document.forms['after'].output.value = '';
		document.forms['after'].url.value = window.location.href.split('?')[0] + '?';
		
		dprint('Input parameters in user units', '')
		dprints('Dname')
		Fs = dprinty('Fs')
		Re = dprinty('Re')
		Qes = dprinty('Qes')
		Qms = dprinty('Qms')
		Vas_liters = dprinty('Vas')
		Sd_cm2 = dprinty('Sd')
		Xmax_mm = dprinty('Xmax')
		Le_mH = dprinty('Le')
		Znom = dprinty('Znom')
		Pe = dprinty('Pe')
		dprints('Bname')
		V_liters = dprinty('V_liters')
		Ndrivers = dprinty('Ndrivers')
		Fport = dprinty('Fport')
		Aport_cm = dprinty('Aport')
		Bport_cm = dprinty('Bport')
		nPorts = dprinty('nPorts')
		endCorrect = dprinty('endCorrect')
		PinExc = dprinty('PinExc')
		
		dprint('Parameters in SI units', '')

		Vas = dprint("Vas = ", Vas_liters*0.001)
		Sd = dprint("Sd = ", Sd_cm2*0.0001)
		Xmax = dprint("Xmax = ", Xmax_mm*0.001)
		Le = dprint("Le = ", Le_mH*0.001)
		Vbox = dprint("Vbox = ", V_liters*0.001)
		Aport = dprint("Aport = ", Aport_cm*0.01)
		Bport = dprint("Bport = ", Bport_cm*0.01)
		Sport = dprint("Sport = ", Aport*Bport*nPorts)
		
		if (Aport < Bport) Rport = dprint("Rport = ", Aport/2)
		else Rport = dprint("Rport = ", Bport/2)

		dprint ('Computed electromechanical parameters in SI units', '')

		ws = dprint("ws, Fs in angular units = ", Fs*2*Math.PI)
		wport = dprint("wport, Fport in angular units = ", Fport*2*Math.PI)
		wp2 = dprint("wport^2 = ", Math.pow(wport,2))
		Km = dprint("Km, spring constant of driver in N/m = ", bpatm*Math.pow(Sd,2)/Vas)
		Rm = dprint("Rm, mechanical damping factor of driver in N s/m = ", Km/Qms/ws)
		Mm = dprint("Mm, mass of driver in kg = ", Km/Math.pow(ws,2))
		BL = dprint("BL, field-length product in Telsa = ", Math.sqrt(Re*Km/Qes/ws))
		Kbox = dprint("Kbox, spring constant of cone pushing box air in N/m = ", Math.pow(Sd,2)*bpatm*Ndrivers/Vbox)
		VinSens = dprint("VinSens, input voltage amplitude for sensitivity graph in V = ", Math.sqrt(PinSens/Ndrivers*Znom))
		VinExc = dprint("VinExc, input voltage amplitude for excursion graph in V = ", Math.sqrt(2*PinExc/Ndrivers*Znom))
		BL2 = dprint("BL^2, square of BL = ", Math.pow(BL,2))
		lport = dprint("lport, length of port in m = ", Sport*bpatm/rho/Vbox/wp2 - Rport*2*endCorrect)
		Lport_cm = dprint("lport, length of port in cm = ", lport*100)
	}

	// Compute excursion for 1 V input signal
	   
	function xexcur(f)
	{
		// angular frequency
		
		w = f*2*Math.PI;
		
		// handle special case if w = wport, to avoid zero divide
		
		if (Math.abs(w - wport) < 0.001) 
		{
			w = w + 0.0001;
		}
		w2 = Math.pow(w,2);
		
		// port factor
		
		pf = w2/(w2 - wp2);
		
		// real part of excursion, generated by computer algebra program
		
		xre = (-Re*(Mm*w2-Kbox*pf-Km)-Le*Rm*w2)*BL
			/(Math.pow(w*BL2-Le*w*(Mm*w2-Kbox*pf-Km)+Re*Rm*w,2)
			+Math.pow(-Re*(Mm*w2-Kbox*pf-Km)-Le*Rm*w2,2));
			
		// imaginary part of excursion, generated by computer algebra program
		
		xim = BL*(-w*BL2+Le*w*(Mm*w2-Kbox*pf-Km)-Re*Rm*w)
			/(Math.pow(w*BL2-Le*w*(Mm*w2-Kbox*pf-Km)+Re*Rm*w,2)
			+Math.pow(-Re*(Mm*w2-Kbox*pf-Km)-Le*Rm*w2,2));
			
		// absolute value of excursion
		
		xabs = Math.sqrt(Math.pow(xre,2) + Math.pow(xim,2));
		
		// convert excursion into mm
		
		xExc = xabs*VinExc*1000;
		
		// compute port air speed
		
		fp = w2/wp2 - 1;
		speed = Math.abs(w*xabs*VinExc*Sd*Ndrivers/Sport/fp/c);
		
		// compute SPL
		
		P1 = 20*Math.log(xabs*Math.abs(VinSens*w2*rho*Sd*
			Ndrivers*pf/2/Dist/Math.PI/splref))/Math.log(10);
			
		P2 = 20*Math.log(xabs*Math.abs(VinExc*w2*rho*Sd*
			Ndrivers*pf/2/Dist/Math.PI/splref))/Math.log(10);
			
		return [xExc, speed, P1, P2];
	}  

	// main line of program
	
	$(function () {
		function plotz()
		{
			getstuff();
			// arrays of data for the graphs
			d1 = []; // gexcursion, peak
			d2 = []; // excursion, waveform based
			s1 = []; // sensitivity in dB SPL at 1 W
			s2 = []; // sensitivity at amplifier power
			m1 = []; // port air speed in Mach
			m2 = []; // port air speed, waveform based
			for (f = 20; f <= 250; f += 1)
			{
				u = xexcur(f);
				d1.push([f, u[0]]);
				m1.push([f, u[1]]);
				s1.push([f, u[2]]);
				s2.push([f, u[3]]);
				u0sum = u[0]/4;
				u1sum = u[1]/4;
				for (j = 2; j <= 4; j++)
				{
					u = xexcur(j*f);
					u0sum = u0sum + u[0]/4;
					u1sum = u1sum + u[1]/4;
				}
				d2.push([f,u0sum]);
				m2.push([f,u1sum]);
				 
			}
			
			d3 = [[20,Xmax_mm],[250,Xmax_mm]];
			xa = {
					transform: function (v) {return Math.log(v)},
					inverseTransform: function (v) {return Math.exp(v)},
					ticks: [20, 30, 40, 50, 60, 80, 100, 120, 150, 200, [250, "250 Hz"]]
					}
					
			$.plot($("#excursion"), 
				[{label: "Peak @ " + PinExc + " W", data: d1},
				{label: "Waveform based @ " + PinExc + " W", data: d2},
				{label: "Xmax", data: d3}],
				{xaxis: xa});
			
			$.plot($("#sensitivity"), 
				[{label: "1 W", data: s1},
				{label: PinExc + " W", data: s2}],
				{xaxis: xa});
				
			$.plot($("#mach"), 
				[{label: "Peak @ " + PinExc + " W", data: m1},
				{label: "Waveform based @ " + PinExc + " W", data: m2}],
				{xaxis: xa});
		}
		
		// assign the above to the re-compute button
		
		$(".stackControls input").click(function () {
			plotz();
		});
		
		// generate first view of plot
		
		plotz();
	});
</script>
<a name="disclaimer"></a><h2>Disclaimer</h2>
Copyright (c) 2014 Francis Deck<br><br>

Permission is hereby granted, free of charge, to any person
obtaining a copy of this software and associated documentation
files (the "Software"), to deal in the Software without
restriction, including without limitation the rights to use,
copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the
Software is furnished to do so, subject to the following
conditions:<br><br>

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.<br><br>

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.
</body>
</html>
