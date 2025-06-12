let player1, player2, ball; // Oyuncular ve top objeleri
let groundY; // Zemin yüksekliği
let score1 = 0, score2 = 0; // Oyuncuların skorları
let goalZoneWidth = 30; // Kale genişliği
let gameOver = false; // Oyun bitiş durumu
let goalCelebrationTimer = 0; // Gol kutlama süresi

function setup() {
  createCanvas(1000, 500); // Oyun alanı boyutu
  groundY = height - 50; // Zeminin alt kenardan uzaklığı
  player1 = new Player(150, groundY, 'a', 'd', 'w', 32, color(255, 215, 0), color(200, 0, 0)); // Oyuncu 1 tanımı
  player2 = new Player(850, groundY, 'ArrowLeft', 'ArrowRight', 'ArrowUp', 73, color(50), color(30)); // Oyuncu 2 tanımı
  ball = new Ball(); // Top nesnesi oluşturuluyor
}

function draw() {
  background(70, 180, 255); // Gökyüzü rengi
  drawField(); // Sahayı çiz
  drawGoals(); // Kaleleri çiz

  if (gameOver) { // Eğer oyun bittiyse
    textSize(48);
    fill(255);
    textAlign(CENTER, CENTER);
    text(score1 > score2 ? "Player 1 Wins!" : "Player 2 Wins!", width / 2, height / 2); // Kazananı yazdır
    return;
  }

  player1.update(); // Oyuncu 1 güncelle
  player2.update(); // Oyuncu 2 güncelle
  ball.update();    // Topu güncelle
  player1.display(); // Oyuncu 1’i çiz
  player2.display(); // Oyuncu 2’yi çiz
  ball.display();    // Topu çiz
  checkCollisions(); // Çarpışma kontrolü yap
  displayScore();    // Skoru göster

  if (goalCelebrationTimer > 0) { // Gol kutlaması varsa
    displayCelebration(); // Kutlamayı göster
    goalCelebrationTimer--;
  }

  if (score1 >= 20 || score2 >= 20) { // 20’ye ulaşan kazanır
    gameOver = true;
  }
}

function drawField() {
  fill(50, 200, 70);
  rect(0, groundY, width, 50); // Zemini çiz
  stroke(255);
  strokeWeight(2);
  line(width / 2, 0, width / 2, height); // Orta çizgi
  noStroke();
}

function drawGoals() {
  fill(220);
  rect(0, groundY - 150, goalZoneWidth, 150); // Sol kale
  rect(width - goalZoneWidth, groundY - 150, goalZoneWidth, 150); // Sağ kale
  fill(180);
  rect(5, groundY - 140, 20, 130); // Sol kale iç çerçevesi
  rect(width - 25, groundY - 140, 20, 130); // Sağ kale iç çerçevesi
}

function displayScore() {
  textSize(32);
  fill(255);
  textAlign(CENTER);
  text(`${score1} - ${score2}`, width / 2, 40); // Skoru ekrana yazdır
}

function displayCelebration() {
  textSize(40);
  fill(255, 255, 0);
  textAlign(CENTER);
  text("GOOOOL!", width / 2, height / 2 - 100); // Gol kutlama yazısı
}

function checkCollisions() {
  let players = [player1, player2]; // Oyuncular listesi
  for (let p of players) {
    let dx = ball.x - p.x;
    let dy = ball.y - (p.y - p.bodyHeight - p.headRadius); // Kafayla top arasındaki mesafe
    let distance = sqrt(dx * dx + dy * dy);
    let minDist = p.headRadius + ball.r;
    if (distance < minDist) { // Kafa çarpışması
      let angle = atan2(dy, dx);
      ball.vx = cos(angle) * 7;
      ball.vy = sin(angle) * 7;
      ball.effectTimer = 10; // Vuruş efekti başlat
    }

    if (p.isKicking) { // Eğer tekme atılıyorsa
      let footY = p.y - 5;
      let footX = p.x + (p.facingRight ? p.bodyWidth / 2 : -p.bodyWidth / 2);
      if (dist(ball.x, ball.y, footX, footY) < ball.r + 10) { // Ayakla çarpışma
        ball.vx += p.facingRight ? 8 : -8;
        ball.vy -= 5;
        ball.effectTimer = 10;
      }
    }
  }

  // Gol olup olmadığını kontrol et
  if (ball.x < goalZoneWidth && ball.y > groundY - 150) { // Sağ oyuncu gol yedi
    score2++;
    ball.reset();
    goalCelebrationTimer = 60;
  } else if (ball.x > width - goalZoneWidth && ball.y > groundY - 150) { // Sol oyuncu gol yedi
    score1++;
    ball.reset();
    goalCelebrationTimer = 60;
  }
}

class Player {
  constructor(x, y, leftKey, rightKey, jumpKey, attackKey, shirtColor, shortColor) {
    this.x = x;
    this.y = y;
    this.vx = 0;
    this.vy = 0;
    this.onGround = true;
    this.keys = { left: leftKey, right: rightKey, jump: jumpKey, attack: attackKey };
    this.bodyHeight = 60;
    this.bodyWidth = 30;
    this.headRadius = 25;
    this.footSize = 10;
    this.isKicking = false;
    this.facingRight = true;
    this.shirtColor = shirtColor;
    this.shortColor = shortColor;
  }

  update() {
    this.vx = 0;
    if (keyIsDown(keyCodeMap(this.keys.left))) {
      this.vx = -5;
      this.facingRight = false;
    }
    if (keyIsDown(keyCodeMap(this.keys.right))) {
      this.vx = 5;
      this.facingRight = true;
    }
    if (keyIsDown(keyCodeMap(this.keys.jump)) && this.onGround) {
      this.vy = -15;
      this.onGround = false;
    }
    this.isKicking = keyIsDown(keyCodeMap(this.keys.attack)); // Tekme tuşuna basılı mı?
    this.x += this.vx;
    this.y += this.vy;
    this.vy += 1;
    if (this.y >= groundY) {
      this.y = groundY;
      this.vy = 0;
      this.onGround = true;
    }
    this.x = constrain(this.x, this.headRadius, width - this.headRadius); // Kenarlara çarpmasın
  }

  display() {
    fill(0);
    let footOffset = this.facingRight ? this.bodyWidth / 2 : -this.bodyWidth / 2;
    rect(this.x + footOffset - 5, this.y - 5, 10, this.footSize); // Ayak

    fill(this.shirtColor);
    rect(this.x - this.bodyWidth / 2, this.y - this.bodyHeight - this.footSize, this.bodyWidth, this.bodyHeight); // Gömlek

    fill(this.shortColor);
    rect(this.x - this.bodyWidth / 2, this.y - this.footSize - 20, this.bodyWidth, 20); // Şort

    fill(255, 220, 180);
    ellipse(this.x, this.y - this.bodyHeight - this.footSize - this.headRadius, this.headRadius * 2); // Kafa

    if (this.isKicking) { // Tekme efekti
      stroke(255, 255, 0);
      strokeWeight(3);
      line(this.x + footOffset, this.y - 10, this.x + footOffset + (this.facingRight ? 20 : -20), this.y - 20);
      noStroke();
    }
  }
}

class Ball {
  constructor() {
    this.r = 20;
    this.reset();
    this.effectTimer = 0;
  }

  reset() {
    this.x = width / 2;
    this.y = groundY - 100;
    this.vx = random([-4, 4]);
    this.vy = 0;
    this.effectTimer = 0;
  }

  update() {
    this.x += this.vx;
    this.y += this.vy;
    this.vy += 0.6; // Yer çekimi
    if (this.y + this.r >= groundY) {
      this.y = groundY - this.r;
      this.vy *= -0.7; // Zıplama etkisi
    }
    if (this.x - this.r < 0 || this.x + this.r > width) {
      this.vx *= -0.9; // Duvara çarpma etkisi
    }
    if (this.effectTimer > 0) {
      this.effectTimer--;
    }
  }

  display() {
    fill(0);
    ellipse(this.x, this.y, this.r * 2); // Topu çiz
    if (this.effectTimer > 0) { // Vuruş efekti varsa çiz
      noFill();
      stroke(255, 255, 0, this.effectTimer * 25);
      strokeWeight(2);
      ellipse(this.x, this.y, this.r * 2 + this.effectTimer * 2);
      noStroke();
    }
  }
}

function keyCodeMap(k) {
  const map = {
    'a': 65,
    'd': 68,
    'w': 87,
    'ArrowLeft': 37,
    'ArrowRight': 39,
    'ArrowUp': 38,
    'i': 73
  };
  return map[k] || k; // Klavye tuşlarını keyCode’a çevirir
}
