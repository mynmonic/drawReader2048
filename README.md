using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class drawReader : MonoBehaviour {

	private GameObject[] tierZero;
	private GameObject[] tierOne;
	private GameObject[] tierTwo;
	private GameObject[] thisTierZero;
	private GameObject newTier;
	private GameObject[] filledSlots;
	private ShopManager shopM;
	private OrderManager orderM;
	private ScoreManager scoreM;
	public bool canSlide;
	public GameObject transParticle;
	public GameObject deliveryTray;
	public GameObject[] guestSeats;
	//private bool haveGemSlided;
	private int slidingGemNum = 0;
	private Vector2 touchStartPos;
	private Vector2 touchMovePos;
	//private float startTouchTime = 0;
	//float movedDistance;
	private float distanceX;
	private float distanceY;
	private float screenDiagonal;
	//private slots[] allSlots;
	//private bool noSlotEmpty;
	private GameObject[][] matrixCells;
	private GameObject thisMatrixCell;
	private GameObject destinationSlot;
	private GameObject activeSlotHandler;
	private int currentFixIndex;		//Fixed row or column index;
	private float combinedID = 0;

	// Use this for initialization
	void Start () {
		//newTier = new GameObject ();
		screenDiagonal = Mathf.Sqrt (Mathf.Pow (Screen.width, 2) + Mathf.Pow (Screen.height, 2));
		//allSlots = FindObjectsOfType(typeof(slots)) as slots[];
		//noSlotEmpty = true;
		shopM = FindObjectOfType<ShopManager> ();
		orderM = FindObjectOfType<OrderManager> ();
		scoreM = FindObjectOfType<ScoreManager> ();
		tierZero = shopM.tierOfZero.ToArray ();
		tierOne = shopM.tierOfOne.ToArray ();
		tierTwo = shopM.tierOfTwo.ToArray ();
		thisTierZero = shopM.currentTierOfZero.ToArray ();
		InitializeMatrix ();
		//destinationSlot = new GameObject();
		SpawnNewGem (5);
		canSlide = true;
		//haveGemSlided = false;
	}
	
	// Update is called once per frame
	void Update () {
		if (canSlide) {
			//touchCount tells how many fingers are touching on the screen
			if (Input.touchCount == 1) {
				Touch touch = Input.GetTouch(0);
				if (touch.phase == TouchPhase.Began) {
					touchStartPos = touch.position;
					//Debug.Log (touchStartPos);
					//startTouchTime = Time.time;	//Used to avoid quick tap. Useless but keep it here for now
				}

				//if (touch.phase == TouchPhase.Moved || touch.phase == TouchPhase.Stationary) {
				if (touch.phase == TouchPhase.Ended) {
					touchMovePos = touch.position;
					if (Vector2.Distance (touchStartPos, touchMovePos)/screenDiagonal > 0.03f) {
						distanceX = touchMovePos.x - touchStartPos.x;
						distanceY = touchMovePos.y - touchStartPos.y;
						MakeDirection ();
					}
					//movedDistance = Vector2.Distance (touchStartPos, touchMovePos)/screenDiagonal;
				}
			}
			//for keyboard test only
			if (Input.GetKeyUp (KeyCode.UpArrow)) {
				distanceX = 0;
				distanceY = 5;
				MakeDirection ();
			} else if (Input.GetKeyUp (KeyCode.RightArrow)) {
				distanceX = 5;
				distanceY = 0;
				MakeDirection ();
			} else if (Input.GetKeyUp (KeyCode.DownArrow)) {
				distanceX = 0;
				distanceY = -5;
				MakeDirection ();
			} else if (Input.GetKeyUp (KeyCode.LeftArrow)) {
				distanceX = -5;
				distanceY = 0;
				MakeDirection ();
			}
		}
	}

	void MakeDirection(){
		canSlide = false;
		if (distanceY > 0 && distanceY > Mathf.Abs (distanceX)) {
			//Go UP
			filledSlots = GameObject.FindGameObjectsWithTag ("haveGem");
			for (int i = 2; i >= 0; i--){		//Start from the third row, one by one to the south
				foreach (GameObject activeSlot in filledSlots) {
					if (activeSlot.GetComponent<slots>().rid == i){		//Filtering the slots only on the current row
						destinationSlot = null;
						combinedID = 0;			//Reset the sum of ID
						currentFixIndex = activeSlot.GetComponent<slots> ().cid;	//Anchoring the column number
						for (int j = i + 1; j <= 3; j++) {		//Scan from next number of current slot row number, one by one to the north
							thisMatrixCell = matrixCells [currentFixIndex][j];		//Fixed x, ascending y
							activeSlotHandler = activeSlot;
							if (thisMatrixCell.transform.childCount == 0) {		//Target slot empty
								destinationSlot = thisMatrixCell;		//Then it will be the destination of sliding
							} else {
								FindingDestination ();
								break;
							}
						}
						if (destinationSlot) {
							GoingDestination ();
							//GoingDestination (new Vector3 (0, 1, 0), "north");
						}
					}
				}
			}
		} else if (distanceX > 0 && distanceX > Mathf.Abs (distanceY)) {
			//Go RIGHT
			filledSlots = GameObject.FindGameObjectsWithTag ("haveGem");
			for (int i = 2; i >= 0; i--){		//Start from the third column, one by one to the left
				foreach (GameObject activeSlot in filledSlots) {
					if (activeSlot.GetComponent<slots>().cid == i){		//Filtering the slots only on the current column
						destinationSlot = null;
						combinedID = 0;			//Reset the sum of ID
						currentFixIndex = activeSlot.GetComponent<slots> ().rid;	//Anchoring the row number
						for (int j = i + 1; j <= 3; j++) {		//Scan from next number of current slot column number, one by one to the right
							thisMatrixCell = matrixCells [j][currentFixIndex];		//Ascending x, fixed y
							activeSlotHandler = activeSlot;
							if (thisMatrixCell.transform.childCount == 0) {		//Target slot empty
								destinationSlot = thisMatrixCell;		//Then it will be the destination of sliding
							} else {
								FindingDestination ();
								break;
							}
						}
						if (destinationSlot) {
							GoingDestination ();
							//GoingDestination (new Vector3 (1, 0, 0), "east");
						}
					}
				}
			}
		} else if (distanceY < 0 && Mathf.Abs (distanceY) > Mathf.Abs (distanceX)) {
			//Go DOWN
			filledSlots = GameObject.FindGameObjectsWithTag ("haveGem");
			for (int i = 1; i <= 3; i++){		//Start from the first row, one by one to the north
				foreach (GameObject activeSlot in filledSlots) {
					if (activeSlot.GetComponent<slots>().rid == i){		//Filtering the slots only on the current row
						destinationSlot = null;
						combinedID = 0;			//Reset the sum of ID
						currentFixIndex = activeSlot.GetComponent<slots> ().cid;	//Anchoring the column number
						for (int j = i - 1; j >= 0; j--) {		//Scan from next number of current slot row number, one by one to the south
							thisMatrixCell = matrixCells [currentFixIndex][j];		//Fixed x, descending y
							activeSlotHandler = activeSlot;
							if (thisMatrixCell.transform.childCount == 0) {		//Target slot empty
								destinationSlot = thisMatrixCell;		//Then it will be the destination of sliding
							} else {
								FindingDestination ();
								break;
							}
						}
						if (destinationSlot) {
							GoingDestination ();
							//GoingDestination (new Vector3 (0, -1, 0), "south");
						}
					}
				}
			}
		} else if (distanceX < 0 && Mathf.Abs (distanceX) > Mathf.Abs(distanceY)) {
			//Go LEFT
			filledSlots = GameObject.FindGameObjectsWithTag ("haveGem");	//Grab all the slots those contain gems
			for (int i = 1; i <= 3; i++){		//Start from the second column, one by one to the right
				foreach (GameObject activeSlot in filledSlots) {
					if (activeSlot.GetComponent<slots>().cid == i){		//Filtering the slots only on the current column
						destinationSlot = null;		//Erase target object container
						combinedID = 0;			//Reset the sum of ID
						currentFixIndex = activeSlot.GetComponent<slots> ().rid;	//Anchoring the row number
						for (int j = i - 1; j >= 0; j--) {		//Scan from next number of current slot column number, one by one to the left
							thisMatrixCell = matrixCells [j][currentFixIndex];		//descending x, fixed y
							activeSlotHandler = activeSlot;
							if (thisMatrixCell.transform.childCount == 0) {		//Target slot empty
								destinationSlot = thisMatrixCell;		//Then it will be the destination of sliding
							} else {
								FindingDestination ();
								break;
							}
						}
						if (destinationSlot) {
							GoingDestination ();
							//GoingDestination (new Vector3 (-1, 0, 0), "west");
						}
					}
				}
			}
		}
		if (slidingGemNum == 0) {
			canSlide = true;
		}
	}

	void FindingDestination(){
		if (thisMatrixCell.transform.childCount == 1) {		//When not empty. If there is no combining in the front.
			//If it has the same gem, go combine with it
			if (thisMatrixCell.transform.GetChild (0).gameObject.name == activeSlotHandler.transform.GetChild (0).gameObject.name) {
				//stacking
				combinedID = activeSlotHandler.transform.GetChild (0).GetComponent<gem> ().ItemID;		//sum ID will remain the same as self.
				destinationSlot = thisMatrixCell;
				//If not same gem, check the followings
			} else if ((thisMatrixCell.transform.GetChild (0).gameObject.tag == "CatA" && activeSlotHandler.transform.GetChild (0).gameObject.tag == "CatB") ||
				(thisMatrixCell.transform.GetChild (0).gameObject.tag == "CatB" && activeSlotHandler.transform.GetChild (0).gameObject.tag == "CatA") ||
				(thisMatrixCell.transform.GetChild (0).gameObject.tag == "CatC" && activeSlotHandler.transform.GetChild (0).gameObject.tag == "CatD") ||
				(thisMatrixCell.transform.GetChild (0).gameObject.tag == "CatD" && activeSlotHandler.transform.GetChild (0).gameObject.tag == "CatC")) {
				//merge and create tier 1 or 2 food
				combinedID = thisMatrixCell.transform.GetChild (0).GetComponent<gem>().ItemID + activeSlotHandler.transform.GetChild (0).GetComponent<gem>().ItemID;
				destinationSlot = thisMatrixCell;
			}
		}
	}
	/*
	void GoingDestination(Vector3 des, string directionToGo){
		activeSlotHandler.tag = "haveNoGem";
		destinationSlot.tag = "haveGem";
		//activeSlot.transform.GetChild (0).transform.parent = destinationSlot.transform;
		//activeSlotHandler.transform.GetChild (0).GetComponent<gem> ().StartMove (des, destinationSlot, combinedID);
		activeSlotHandler.transform.GetChild (0).GetComponent<gem> ().StartMove (destinationSlot, combinedID, directionToGo);
		activeSlotHandler.transform.GetChild (0).transform.parent = destinationSlot.transform;
		//destinationSlot.GetComponent<slots> ().incomingGem += 1;
		haveGemSlided = true;
	}*/

	void GoingDestination(){
		slidingGemNum++;
		activeSlotHandler.tag = "haveNoGem";
		destinationSlot.tag = "haveGem";
		//activeSlot.transform.GetChild (0).transform.parent = destinationSlot.transform;
		//activeSlotHandler.transform.GetChild (0).GetComponent<gem> ().StartMove (des, destinationSlot, combinedID);
		activeSlotHandler.transform.GetChild (0).GetComponent<gem> ().StartMove (destinationSlot, combinedID);
		activeSlotHandler.transform.GetChild (0).transform.parent = destinationSlot.transform;
		//destinationSlot.GetComponent<slots> ().incomingGem += 1;
		//haveGemSlided = true;
	}

	public void SlidingEndCheck(){
		slidingGemNum--;
		if (slidingGemNum <= 0) {
			SpawnNewGem (1);
			slidingGemNum = 0;
			StartCoroutine (NextStep ());
			//canSlide = true;
		}
	}

	IEnumerator NextStep(){
		yield return new WaitForSeconds (0.1f);
		canSlide = true;
	}

	void SpawnNewGem(int amount){
		for (int i = 0; i < amount; i++) {
			GameObject[] unfilledSlot = GameObject.FindGameObjectsWithTag ("haveNoGem");
			if (unfilledSlot.Length > 0) {
				int rand = (int)Random.Range (0, unfilledSlot.Length);
				GameObject newGemArrival = Instantiate (thisTierZero [Random.Range (0, thisTierZero.Length)], unfilledSlot [rand].transform.position, Quaternion.identity);
				newGemArrival.transform.parent = unfilledSlot [rand].transform;
				newGemArrival.transform.position = unfilledSlot [rand].transform.position;
				unfilledSlot [rand].tag = "haveGem";
			}
		}
		//haveGemSlided = false;
	}

	void InitializeMatrix(){
		matrixCells = new GameObject[][] {
			new GameObject[4],
			new GameObject[4],
			new GameObject[4],
			new GameObject[4]
		};
		matrixCells [0] [0] = GameObject.Find ("Slot-1");
		matrixCells [0] [1] = GameObject.Find ("Slot-5");
		matrixCells [0] [2] = GameObject.Find ("Slot-9");
		matrixCells [0] [3] = GameObject.Find ("Slot-13");
		matrixCells [1] [0] = GameObject.Find ("Slot-2");
		matrixCells [1] [1] = GameObject.Find ("Slot-6");
		matrixCells [1] [2] = GameObject.Find ("Slot-10");
		matrixCells [1] [3] = GameObject.Find ("Slot-14");
		matrixCells [2] [0] = GameObject.Find ("Slot-3");
		matrixCells [2] [1] = GameObject.Find ("Slot-7");
		matrixCells [2] [2] = GameObject.Find ("Slot-11");
		matrixCells [2] [3] = GameObject.Find ("Slot-15");
		matrixCells [3] [0] = GameObject.Find ("Slot-4");
		matrixCells [3] [1] = GameObject.Find ("Slot-8");
		matrixCells [3] [2] = GameObject.Find ("Slot-12");
		matrixCells [3] [3] = GameObject.Find ("Slot-16");
	}

	public void Reset(){
		for (int i = 0; i <= 3; i++) {
			for (int j = 0; j <= 3; j++) {
				matrixCells [i] [j].tag = "haveNoGem";
				if (matrixCells [i] [j].transform.childCount > 0) {
					Destroy (matrixCells [i] [j].transform.GetChild (0).gameObject);
					//matrixCells [i] [j].GetComponent<slots> ().incomingGem = 0;
				}
			}
		}
		combinedID = 0;
		orderM.ResetOrder ();
		scoreM.ResetScore ();
		//noSlotEmpty = true;
		SpawnNewGem (5);
		canSlide = true;
		slidingGemNum = 0;
		//haveGemSlided = false;
	}

	public void TransformNewTier(float a, int b, GameObject c){	//a for itemID, b for stack number
		//GameObject newTier = new GameObject ();
		//Debug.Log ("Received request ID is " + a);

		if (a == 1.1f) {
			newTier = Instantiate (tierOne [0], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 1.2f) {
			newTier = Instantiate (tierOne [1], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 1.4f) {
			newTier = Instantiate (tierOne [2], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 2.1f) {
			newTier = Instantiate (tierOne [3], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 2.2f) {
			newTier = Instantiate (tierOne [4], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 2.4f) {
			newTier = Instantiate (tierOne [5], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 1.11f) {
			newTier = Instantiate (tierTwo [0], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 1.21f) {
			newTier = Instantiate (tierTwo [1], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 1.41f) {
			newTier = Instantiate (tierTwo [2], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 2.11f) {
			newTier = Instantiate (tierTwo [3], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 2.21f) {
			newTier = Instantiate (tierTwo [4], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 2.41f) {
			newTier = Instantiate (tierTwo [5], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 1.12f) {
			newTier = Instantiate (tierTwo [6], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 1.22f) {
			newTier = Instantiate (tierTwo [7], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 1.42f) {
			newTier = Instantiate (tierTwo [8], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 2.12f) {
			newTier = Instantiate (tierTwo [9], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 2.22f) {
			newTier = Instantiate (tierTwo [10], c.transform.position, Quaternion.identity) as GameObject;
		} else if (a == 2.42f) {
			newTier = Instantiate (tierTwo [11], c.transform.position, Quaternion.identity) as GameObject;
		} else {
			print ("Something wrong with the dish. The request ID is " + a);
		}

		if (newTier) {
			Instantiate (transParticle, c.transform.position, Quaternion.identity);
			GameObject thisGuestSeat;
			thisGuestSeat = orderM.checkOrderMatch (a);
			if (thisGuestSeat != null) {
				//Play delivering animation here!
				scoreM.UpdateIncome (FindTierLevel (a) * shopM.UnitPrice * b);
				scoreM.LikeProgressing (FindTierLevel (a));
				c.tag = "haveNoGem";
				GameObject thisDeliveryTray = Instantiate (deliveryTray, c.transform.position, Quaternion.identity) as GameObject;
				newTier.transform.parent = thisDeliveryTray.transform;
				newTier.transform.GetChild (0).gameObject.SetActive (false);
				thisDeliveryTray.GetComponent<DeliveryTray> ().GetTarget (thisGuestSeat);

			} else {
				newTier.transform.parent = c.transform;
				newTier.GetComponent<gem> ().StackNumber = b;
			}
		}

		//newTier.GetComponent<gem> ().SetStackNumber ();
		//Debug.Log("The spawned ID is " + newTier.GetComponent<gem>().ItemID);
	}

	public GameObject CheckExistingFood(float tmp){
		int thisTierLevel = FindTierLevel (tmp);
		if (thisTierLevel > 1) {	//check tier 2
			filledSlots = GameObject.FindGameObjectsWithTag ("haveGem");
			foreach (GameObject activeSlot in filledSlots) {
				if (activeSlot.transform.GetChild (0).GetComponent<gem> ().ItemID == tmp) {
					scoreM.UpdateIncome (thisTierLevel * shopM.UnitPrice * activeSlot.transform.GetChild (0).GetComponent<gem> ().StackNumber);
					scoreM.LikeProgressing (thisTierLevel);
					//Destroy (activeSlot.transform.GetChild (0).gameObject);
					activeSlot.tag = "haveNoGem";
					return activeSlot.transform.GetChild (0).gameObject;
				}
			}
		} else {	//check tier 1
			filledSlots = GameObject.FindGameObjectsWithTag ("haveGem");
			foreach (GameObject activeSlot in filledSlots) {
				if (activeSlot.transform.GetChild (0).GetComponent<gem> ().ItemID == tmp) {
					activeSlot.transform.GetChild (0).GetComponent<gem> ().SetSellable ();
				}
			}
		}
		return null;
	}

	public int FindTierLevel(float tmp){
		string tmpString = tmp.ToString ();
		if (tmpString.Length == 3) {
			return 1;
		} else {
			return 2;
		}
	}
}
