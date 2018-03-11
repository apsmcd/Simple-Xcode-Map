# Simple-Xcode-Map
Xcode script for a map, user's location, and a search bar

~~~~
import UIKit
import MapKit
import CoreLocation

class ViewController: UIViewController, MKMapViewDelegate, CLLocationManagerDelegate, UISearchBarDelegate {
    @IBOutlet var map: MKMapView!

    @IBAction func searchButton(_ sender: Any) {
        let searchController = UISearchController(searchResultsController: nil)
        searchController.searchBar.delegate = self
        present(searchController, animated: true, completion: nil)
    }

    func searchBarSearchButtonClicked(_ searchBar: UISearchBar) {
        // ignoring user
        UIApplication.shared.beginIgnoringInteractionEvents()

        //Activity indicator
        let activityIndicator = UIActivityIndicatorView()
        activityIndicator.activityIndicatorViewStyle = UIActivityIndicatorViewStyle.gray
        activityIndicator.hidesWhenStopped = true
        activityIndicator.startAnimating()

        self.view.addSubview(activityIndicator)

        //hide search bar
        searchBar.resignFirstResponder()
        dismiss(animated: true, completion: nil)

        //create the search request
        let searchRequest = MKLocalSearchRequest()
        searchRequest.naturalLanguageQuery = searchBar.text

        let activeSearch = MKLocalSearch(request: searchRequest)

        activeSearch.start {(response, error) in

            activityIndicator.stopAnimating()
            UIApplication.shared.endIgnoringInteractionEvents()

            if response == nil {
                print("error")
            }
            else {
                //remove annotations
                let annotations = self.map.annotations
                self.map.removeAnnotations(annotations)

                //getting data
                let latitude = response?.boundingRegion.center.latitude
                let longitude = response?.boundingRegion.center.longitude

                //create annotations
                let annotation = MKPointAnnotation()
                annotation.title = searchBar.text
                annotation.coordinate = CLLocationCoordinate2DMake(latitude!, longitude!)
                self.map.addAnnotation(annotation)

                //zooming in on search
                let coordinate:CLLocationCoordinate2D = CLLocationCoordinate2DMake(latitude!, longitude!)
                let span = MKCoordinateSpanMake(0.1, 0.1)
                let region = MKCoordinateRegionMake(coordinate, span)
                self.map.setRegion(region, animated: true)
            }
        }

    }

    let locationManager = CLLocationManager()
    var movedToUserLocation = false

    @objc func dismissKeyboard() {
        view.endEditing(true)
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.

        let keyboardDismiss = UITapGestureRecognizer(target: self, action: #selector(self.dismissKeyboard))
        view.addGestureRecognizer(keyboardDismiss)

        map.delegate = self
        locationManager.delegate = self

        locationManager.desiredAccuracy = kCLLocationAccuracyBest
    }

    @IBAction func navBar(_ sender: Any) {
    }

    func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
        switch status {
        case .denied, .restricted:
            print("Uh oh")
        case .notDetermined:
            manager.requestWhenInUseAuthorization()
        default:
            manager.startUpdatingLocation()
        }
    }

    func map( map: MKMapView, didUpdate userLocation: MKUserLocation){
        if !movedToUserLocation {
            map.region.center = map.userLocation.coordinate

            movedToUserLocation = true
        }
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
}
~~~~
