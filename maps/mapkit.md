theme: Plain jane, 3
autoscale: true
footer: (c) Diego Freniche / @dfreniche / www.freniche.com

# Core Networking

# MapKit

---

## Link MapKit framework

Project > Target > Capabilities > Maps

In code import frameworks

```swift
import MapKit
import CoreLocation

```

---


## Obtain our position


```swift
locationManager = CLLocationManager()
// ask for Location authorization
locationManager?.requestWhenInUseAuthorization()
```

- we need to add this Key to our PList file

```
<key>NSLocationWhenInUseUsageDescription</key>
	<string>This App needs to access your location</string>
```

---

## Center the map

```swift
let londonLocation = CLLocation(latitude: 51.509865, longitude: -0.118092)
// self.mapView.setCenter(londonLocation.coordinate, animated: true)

let region = MKCoordinateRegion(center: londonLocation.coordinate, span: MKCoordinateSpanMake(0.2, 0.2))
self.mapView.setRegion(region, animated: true)
```


---

## CLLocation delegate


```swift
... // viewDidLoad ?
	locationManager = CLLocationManager()
	locationManager?.requestWhenInUseAuthorization()
	locationManager?.delegate = self
	locationManager?.startUpdatingLocation()
...

func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    print(locations.first.debugDescription)
}

```


---

## MapView delegate

- Conform to MKMapViewDelegate
- to add pins to the map


```swift
// Tells the delegate that the map view is about to start rendering some of its tiles
func mapViewWillStartRenderingMap(_ mapView: MKMapView) {
	print("Start rendering")
}

func mapViewDidFinishRenderingMap(_ mapView: MKMapView, fullyRendered: Bool) {
	print("Finish rendering")
}

// Tells the delegate that the specified map view is about to retrieve some map data
func mapViewWillStartLoadingMap(_ mapView: MKMapView) {
	print("Start loading")
}

func mapViewDidFinishLoadingMap(_ mapView: MKMapView) {
	print("Finish loading")
}
```

---

## Adding pins to the map (Annotations)

1. Create a class that conforms to NSObject & MKAnnotation

```swift
class MapPin: NSObject, MKAnnotation {
    var coordinate: CLLocationCoordinate2D
    var title: String?
    var subtitle: String?
    
    init(coordinate: CLLocationCoordinate2D) {
        self.coordinate = coordinate
    }
}

```

---

## Adding pins to the map (Annotations)

2. Added mapview delegate: MapView will paint annotations only when on screen


```swift
func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
	// Don't want to show a custom image if the annotation is the user's location.
	guard !(annotation is MKUserLocation) else {
		return nil
	}
	
	// Better to make this class property
	let annotationIdentifier = "AnnotationIdentifier"
	
	var annotationView: MKAnnotationView?
	if let dequeuedAnnotationView = mapView.dequeueReusableAnnotationView(withIdentifier: annotationIdentifier) {
		annotationView = dequeuedAnnotationView
		annotationView?.annotation = annotation
	}
	else {
		annotationView = MKAnnotationView(annotation: annotation, reuseIdentifier: annotationIdentifier)
		annotationView?.rightCalloutAccessoryView = UIButton(type: .detailDisclosure)
	}
	
	if let annotationView = annotationView {
		// Configure your annotation view here
		annotationView.canShowCallout = true
		annotationView.image = UIImage(named: "marker")
	}
	
	return annotationView
}
```

---

## Adding pins to the map (Annotations)

3. Add some pins

```swift
let a1 = MapPin(coordinate: CLLocationCoordinate2D(latitude: 51.509865, longitude: -0.1180))
a1.title = "Pin 1"
a1.subtitle = "Subtitle 1"

self.mapView.addAnnotation(a1)
```

---

## Fit all annotations on mapView


```objectivec
- (void)zoomToFitMapAnnotations:(MKMapView *)mapView {
    if ([mapView.annotations count] == 0) return;
    
    CLLocationCoordinate2D topLeftCoord;
    topLeftCoord.latitude = -90;
    topLeftCoord.longitude = 180;
    
    CLLocationCoordinate2D bottomRightCoord;
    bottomRightCoord.latitude = 90;
    bottomRightCoord.longitude = -180;
    
    for(id<MKAnnotation> annotation in mapView.annotations) {
        if(annotation == mapView.userLocation) {
            continue;
        }

        topLeftCoord.longitude = fmin(topLeftCoord.longitude, annotation.coordinate.longitude);
        topLeftCoord.latitude = fmax(topLeftCoord.latitude, annotation.coordinate.latitude);
        bottomRightCoord.longitude = fmax(bottomRightCoord.longitude, annotation.coordinate.longitude);
        bottomRightCoord.latitude = fmin(bottomRightCoord.latitude, annotation.coordinate.latitude);
    }
    
    MKCoordinateRegion region;
    region.center.latitude = topLeftCoord.latitude - (topLeftCoord.latitude - bottomRightCoord.latitude) * 0.5;
    region.center.longitude = topLeftCoord.longitude + (bottomRightCoord.longitude - topLeftCoord.longitude) * 0.5;
    
    // Add a little extra space on the sides
    region.span.latitudeDelta = fabs(topLeftCoord.latitude - bottomRightCoord.latitude) * 1.5;
    region.span.longitudeDelta = fabs(bottomRightCoord.longitude - topLeftCoord.longitude) * 1.5;
    
    region = [mapView regionThatFits:region];
    [mapView setRegion:region animated:YES];
}


```