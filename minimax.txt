let data=0
var strtbrd;
const human= 'O';
const computer= 'X';
//all the winning combo arrays
const wincoms = [
    [0,1,2,],
    [0,4,8],
    [0,3,6],
    [1,4,7],
    [2,5,8],
    [2,4,6],
    [3,4,5],
    [6,7,8],
]

const cells = document.querySelectorAll('.cell');
//this means the cells variable will store a reference of each element with the class='cell' in the html

play();
function play(){
    document.querySelector(".endgame").style.display = "none" //to make the you win plaque disappear when play is clicked
    strtbrd= Array.from(Array(9).keys()) //this will create an array with 9 elements which will contain only the keys of the elements
    for(var i=0;i<cells.length;i++){
        cells[i].innerText='';
        cells[i].style.removeProperty('background-color'); //removes the highlight of the selected blocks from the previous game
        cells[i].addEventListener('click',turnClick,false); //makes the function listen for the click which will then call the turnclick function
    }
}

function turnClick(square){
    if (typeof strtbrd[square.target.id] == 'number') {//condition to prevent making plays on spots where there already is a move made
        turn(square.target.id,human)
        //this function calls the turn function when the human player clicks the square
            if(!checktie()) turn(bestspt(),computer);//condition to check if there is a tie
    } 
}

function turn(squareId,player){
    strtbrd[squareId] = player; //its gonna show which player took a turn on the specific square
    document.getElementById(squareId).innerText = player; //to display the text when the square is clicked
    //it gets the square id and updates the display according to the player
    let gamewon = checkwin(strtbrd,player)
    if(gamewon) gameover(gamewon)
}

function checkwin(board, player){
    let moves = board.reduce((a,e,i) => (e == player)? a.concat(i) : a, [])
    //moves means every spot where the player has played a move
    //board- substitute array for the the origboard array
    //reduce- function that will go through each element of the board array and will return a single value
    //a- accumulator,the value that is going to be given back at the end, initialised with an empty array
    //e-element of the board array on which the function is being used
    //i-index
    //working- if the element(e) is equal to player, the index(i) will be added to the accumulator array
    //if not equal then the accumulator will be returned just as it was and nothing will be added to it
    //this is just a way to find every index on which the player has already played in
    let gamewon = null; //declaration of gamewon
    for (let[index, win] of wincoms.entries()){//wincombo.entries is a way to get the index and the win condition
        //the index variable is going to keep the index of the variable and the win is going to loop through the arrays to find the win condition
        //win is the variable where each array of the wincoms is going to be stored and checked for the win condition
        if (win.every(elem => moves.indexOf(elem) > -1)){
        //this means that for every element in the current win array we are going to check if all the places the player has played on the board is more than -1
        //fancy way of saying that, has the player played in every spot that counts as a win for that 'win'
            //win.every means the condition is going through each element of the current win array
            //Ex- the first win is going to be [0,1,2], so win.every will go through 0,1 and 2 to check for the condition
            gamewon={index: index, player:player};
            //from here we know from which combo the player won and which player won
            break;
        }
    }
    return gamewon;
}

function gameover(gamewon) {
    if (gamewon.player == human) {//condition to update the scorepoint
        data=data+1;
        document.getElementById("count").textContent = data
    }
    for (let index of wincoms[gamewon.index]){
        //in this loop we going to go through each index of the win combo and execute the following
        document.getElementById(index).style.backgroundColor =
        gamewon.player == human ? "rgb(74, 74, 231,.5)" : "rgb(246, 64, 64,.5)";
        //if the game is won by the human player the win squares are going to be blue, if the ai wins then its going to be red
    }
    for(var i=0; i<cells.length; i++){//loop to stop further moves from being played after the game has been won
        cells[i].removeEventListener('click', turnClick, false);
    }
    declarewin(gamewon.player == human ? "You Win" : "You Lose!");
}

function declarewin(who) {
    document.querySelector(".endgame").style.display = "block";
    document.querySelector(".endgame .text").textContent = who;
}

function emptysqr() { //function to find out the empty squares and return them
    return strtbrd.filter(s => typeof s =='number');
    //it is going to filter the squares of the board and if the square is a number,i.e, not filled with 'o' or 'x', return them
}

function bestspt() {
    return minimax(strtbrd,computer).index;
//.index is used as the return value of minimax is going to be an object and .index is going to be the index with the computer's moves
}

function checktie() {
    if (emptysqr().length == 0){ //the condition inside means every square has filled up and no winners yet as it checks every time anyone makes a turn
        for(var i=0; i<cells.length; i++){
            cells[i].style.backgroundColor = "rgb(52, 170, 52,.5)";
            cells[i].removeEventListener('click', turnClick, false);  
        }
        declarewin("It's a Tie!!")
        return true;
    }
    return false;
}

function minimax(newbrd,player) {
    var availspots = emptysqr();//finds the indexes of available spots and sets them to availspots
    
    if (checkwin(newbrd,human)) {
        return {points: -10};
    } else if (checkwin(newbrd,computer)) {
        return {points: 10};
    } else if (availspots.length === 0) {
        return {points: 0};
    }

    var movesets = [];
    for (var i = 0; i<availspots.length ; i++) {
        var movenum = {};//object to collect each moves's index and points
        movenum.index = newbrd[availspots[i]];// setting the index number of the empty spot that was stored as a number from the strtbrd to the index property of moveum object
        newbrd[availspots[i]] = player; // set the empty spot on the newbrd to the current player

        //calling the minimax function with the other player and the newly changed newbrd
        if(player == computer) {
            var result = minimax(newbrd,human);
            movenum.points = result.points; // storing the object resulting from minimax function call with a points property to points property of the move object
        } else { //if the minimax function does not find a terminal state it keeps on going lvl by lvl deeper into the game
            var result = minimax(newbrd,computer);
            movenum.points = result.points; 
        }//this recursion happens until it reaches a terminal state and returns points one level up

        newbrd[availspots[i]] = movenum.index;//resetting the newbrd to its original state
        movesets.push(movenum); //pushing the movenum object to movesets array

    }

    var bestmv; //for evaluating the best move in the movesets array for the ai and the lowest point for the human
    if(player === computer) {
        var bestpoint = -10000;
        for(var i= 0; i < movesets.length;i++) {//starts with a very low number to loop through the entire array
            if(movesets[i].points > bestpoint) {//if the move has higher points than the bestpoint then the algo stores that move. For similar points only the first move is stored
                bestpoint = movesets[i].points;
                bestmv = i;
            }
        }
    } else {
        var bestpoint = 10000;
        for(var i= 0; i < movesets.length; i++) {//starts with a very high number to loop through the entire array
            if(movesets[i].points < bestpoint) {//if the move has lower points than the bestpoint then the algo stores that move. For similar points only the first move is stored
                bestpoint = movesets[i].points;
                bestmv = i;
            }
        }
    }
    return movesets[bestmv];
}

function refresh() {
    location.reload();
}