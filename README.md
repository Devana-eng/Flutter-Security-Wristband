import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_database/firebase_database.dart';
import 'package:firebase_storage/firebase_storage.dart';
import 'package:audio players/audio players.dart';

void main() async {
WidgetsFlutterBinding.ensureInitialized();
await Firebase.initializeApp();
runApp(SecurityApp());
}

class SecurityApp extends StatelessWidget {
@override
Widget build(BuildContext context) {
return MaterialApp(
debugShowCheckedModeBanner: false,
home: HomeScreen(),
);
}
}

class HomeScreen extends StatefulWidget {
@override
_HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
GoogleMapController? _mapController;
LatLng _currentLocation = LatLng(0.0, 0.0);
final DatabaseReference _database = FirebaseDatabase.instance.ref();
final AudioPlayer _audioPlayer = AudioPlayer();
String? _audioUrl;

@override
void initState() {
super.initState();
_fetchLocation();
_fetchAudio();
}

void _fetchLocation() {
_database.child("location").onValue.listen((event) {
var snapshot = event.snapshot.value as Map<dynamic, dynamic>?;
if (snapshot != null) {
double lat = snapshot["latitude"];
double lng = snapshot["longitude"];
setState(() {
_currentLocation = LatLng(lat, lng);
});
}
});
}

void _fetchAudio() async {
final ref = FirebaseStorage.instance.ref().child("recordings/emergency_audio.mp3");
String url = await ref.getDownloadURL();
setState(() {
_audioUrl = url;
});
}

void _playAudio() {
if (_audioUrl != null) {
_audioPlayer.play(UrlSource(_audioUrl!));
}
}

@override
Widget build(BuildContext context) {
return Scaffold(
appBar: AppBar(title: Text("Security Wristband App")),
body: Column(
children: [
Expanded(
child: GoogleMap(
initialCameraPosition: CameraPosition(
target: _currentLocation,
zoom: 15,
),
onMapCreated: (controller) => _mapController = controller,
markers: {
Marker(
markerId: MarkerId("emergencyLocation"),
position: _currentLocation,
infoWindow: InfoWindow(title: "Emergency Location"),
),
},
),
),
SizedBox(height: 20),
ElevatedButton(
onPressed: _playAudio,
child: Text("Play Emergency Audio"),
),
SizedBox(height: 20),
],
),
);
}
}
